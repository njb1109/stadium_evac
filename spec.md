## Stadium Evacuation

## Purpose
A modeling app used by facilities managers and emergency responders. The user inputs how many occupants are in the Marriott Center at BYU, and the app evaluates and animates how fast the occupants can evacuate in the case of an emergency.

## Inputs - every value the user enters, with units, ranges, defaults, and the control used to enter it
-User inputs the current occupancy of the Marriott Center from 100-2000 people.
-User inputs any obstacles are present, such as rows of seats

## Calculations - the equations, rules, or logic the app uses (formulas, thresholds, lookup tables, how downloaded data is processed), with every variable and its units defined
-Rate of evacuation = 24 people/minute/door width in feet
-Total evacuation time = Current occupancy/Rate of evacuation + Time to get to doors

## Outputs - what the app shows: numbers with units, tables, plots, and diagrams
-Time in hours, minutes, seconds
-Plot of people evacuated by time taken
-Animation of people evacuating on the Marriott Center floor plan

## Look and feel - the layout, what the app shows when it first opens, and how it works on a phone
-Opens with a default occupancy of 500 people
-Clean and professional. One light color used throughout along with a grey color.
-Floor plan should be the first thing that shows up with panels displaying statistics and calculations below it.

## Validation - at least one case worked by hand, with exact inputs and expected outputs
For a crowded room with one door, the time to empty is controlled by how fast people can pass through the door. A commonly used maximum flow through a door is about 24 people per minute per foot of door width (from the SFPE Handbook of Fire Protection Engineering). Compute how long 500 people need to get through a 6 ft door including how long it takes people to get to the door.

## Out of scope - what the app will not do
-Saving specific user inputs and settings
-Specific cases, such as accessibility requirements
