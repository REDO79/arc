# arc
Creating and offseting arcs by rotation and overlapping in dynamo
import sys
import clr
import math
clr.AddReference('ProtoGeometry')
from Autodesk.DesignScript.Geometry import *
#Circle inputs
circle_curve = IN[0]
offset_distance = IN[1]
rebar_Diameter = IN[2]

overlap_length = 50 * rebar_Diameter / 1000
# Circle's center point
Center_point = circle_curve.CenterPoint
radius = circle_curve.Radius
circle = Circle.ByCenterPointRadius(Center_point, radius)
Start_pt1 = circle.StartPoint
End_pt1 = circle.PointAtSegmentLength(12)
arc = Arc.ByCenterPointStartPointEndPoint(Center_point, Start_pt1, End_pt1)
Radius = Line.ByStartPointEndPoint(Start_pt1, Center_point)

# vector to offset circles inwards by offset_distance
offset_vector = Radius.Direction.Normalized()
Normal_vector = Vector.ByCoordinates(0, 0, 1)
lst = [circle]
lst1 = [arc]
#Offset circles inwards
k =0
d = 0
while k < 600:
    k +=1
    d += offset_distance
    circle_offset = circle.Offset(-d)
    try:
        Radius_offset = circle_offset.Radius
    except:
        circle_offset = circle_offset.Explode()[0] 
    Start_pt1 = circle_offset.StartPoint     
    End_pt1 = circle_offset.PointAtSegmentLength(12)
    if circle_offset.Length< 12:
        break
    arc = Arc.ByCenterPointStartPointEndPoint(Center_point, Start_pt1, End_pt1)
    # filling in arcs in lst1 
    lst1.append(arc)
out = []    
for i in lst1:
    temp = []
    Start_Angle = i.StartAngle
    End_Angle = Start_Angle + i.SweepAngle
    temp.append(i)
    k = 0 
    while k < 500:
        k += 1
        overlap_Angle = (overlap_length * 360) / (2 * math.pi * i.Radius)
        # compute remaining arc
        rest_Arc = Arc.ByCenterPointRadiusAngle(Center_point, i.Radius, End_Angle - overlap_Angle, Start_Angle + overlap_Angle, Normal_vector)
        if math.ceil(rest_Arc.Length) < 12:
            temp.append(rest_Arc)
            #temp = [c for c in temp if c.Length > overlap_length]
            out.append(temp)
            break
        
        #rotate arcs    
        i = i.Rotate(Center_point, Normal_vector, i.SweepAngle - overlap_Angle)
        #update End_Angle
        End_Angle = i.StartAngle + i.SweepAngle
        temp.append(i)
        
OUT = out
