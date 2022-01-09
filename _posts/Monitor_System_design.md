
# Overview
## What kind of problem Incident Platform Solve
For example, there is an online payment company which provides payment services for merchants and customers. The company categorizes all businesses into offline business, online business. 
And offline business can be categorized into:
 - restaurants
 - supermarkets  
 - schools
 - hospitals, etc. 
Online business can be categoried into: 
 - E-commerce business
 - E-Game
 - Tiket and booking
 - Traveling, etc.
In each category there are many merchants, each of merchants built an app to provide services, and customers pays with the company's app to buy the service or products.
But sometimes, the merchants' apps will fail to load, or the customers can't login to the app, or the payment does't work. Each these failure will cause the lose of
money and the patience of customers. In order to solve these problems, the company need to build a incident platform.

## The Platform defination
The business goal of Incident Platform: Improve the Stability of the Whole System（稳定性）.
There are various ways to calculate Stability:
 - UV Stability = 1 - ImpactUsers/WholeUsers, for online business (影响用户数)
 - PV Stability = 1 - ImpactPv/WholePv, for web services, some pages work, some not （影响服务次数）
 - Time Stability = 1 - NotWorkingTime/WholeTime, for hardware （影响时长）

In practice, the first one is the mostly chosed.

In order to achieve the goal, there are 2 sub KPIs:
 - Reduce the number of incidents (故障数). The ways:
  - Pre-Online Test: Performance Test
  - Periodic Risk Inspection: Daily or weekly inspect the system based on risk rules
  - For Security: Red Army vs Blue Army, White Hat Bonus, etc
 - Reduce the average time cost of incidents（平均故障时长）
  - Build emergency plan to Standardize incident handling processes
  - Tools to help automatically handling process: Auto Alert, Auto call people, Auto create groups, etc.

In practice, the second way to more reliable and predictable than the first one.

## International Standard
ITIL V3.0 (Information Technology Infrastructure Library) By IBM, HP, Government, etc.
The part of Incident Management in ITIL.


