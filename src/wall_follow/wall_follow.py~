#!/usr/bin/env python

#wall follow behavior


import rospy
from math import cos
from std_msgs.msg import String
from geometry_msgs.msg import Twist
from geometry_msgs.msg import Vector3
from sensor_msgs.msg import LaserScan

lowTolFac = 0.9
highTolFac = 1.1
turnDirect = 0
velocity = 0

pub = rospy.Publisher('/cmd_vel', Twist, queue_size=10)
rospy.init_node('wallFollow', anonymous=True)
r = rospy.Rate(10) # 10hz

def obstacleAvoidance(msg):
	"""I'm the one who makes you not crash"""
	pub = rospy.Publisher('/cmd_vel', Twist, queue_size=10)
	for index, value in enumerate(msg.ranges):
		if (index < 60 or index > 300) and 0 < value < 1:
			print "AVOID!!!"
			pub.publish(linear=Vector3(x=-1),angular=Vector3(z=-1))
			rospy.Rate(1).sleep()
			return False
		else:
			return True
	

def wallDetect(msg):
	"""Takes laser data to identify and locate wall, and determines direction to turn"""
	#is there the correct amount of data? Is there no obstacle?
	global turnDirect
	if len(msg.ranges) == 360:
		minDist = 10
		angInd = -1
		velocity = 0
		for index, value in enumerate(msg.ranges):
			if 0 < value < minDist:
				minDist = value
				angInd = index
		if obstacleAvoidance(msg):
			hypoLow = lowTolFac * (minDist * cos(0.0872664626))
			hypoHigh = highTolFac * (minDist * cos(0.0872664626))
			#check if data is 90* to sensor - if so it's a wall
			upperAngle = angInd -5
			lowerAngle = angInd +5
			if lowerAngle < 0:
				lowerAngle = lowerAngle + 359
			if upperAngle > 359:
				upperAngle = upperAngle - 359
			if msg.ranges[lowerAngle] > hypoLow and msg.ranges[lowerAngle] < hypoHigh and msg.ranges[upperAngle] > hypoLow and msg.ranges[upperAngle] < hypoHigh:
				if 0 <= angInd < 90 or 180 < angInd < 270:
					#make right turn
					if angInd < 90:
						turnDirect = -((90-angInd)/90.0)
					else:
						turnDirect = -((270 - angInd)/90.0)
						print "it's here!"
						print turnDirect

				elif 90 < angInd < 180 or 270 < angInd <= 359:
					#make left turn
					if angInd < 180:
						turnDirect = ((180 - angInd)/90.0)
					else:
						turnDirect = ((360 - angInd)/90.0)
				else:
					turnDirect = 0
			else:
				turnDirect = 0
		else:
			turnDirect = 1
			velocity = -1

 

		

def wallFollow():
	"""Detect wall and informs of the angle"""
	sub = rospy.Subscriber('/scan', LaserScan, wallDetect)
	while not rospy.is_shutdown():
		#this is close enought to parellel
		if abs(turnDirect) < 0.1:
			msg = Twist(linear=Vector3(x = .3))
			print "close to parallel or searching"
		else:
			msg = Twist(linear=Vector3(x=0),angular=Vector3(z=turnDirect))
			print "not close to parellel:", turnDirect
        	pub.publish(msg)
        	r.sleep()

if __name__ == '__main__':
    try:
        wallFollow()
    except rospy.ROSInterruptException: pass


