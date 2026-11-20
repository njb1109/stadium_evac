## Stadium Evacuation

## Purpose
A planning and training app for facilities managers, building operators, event staff, emergency responders, and planning personnel. It models the Marriott Center floor plan and estimates the total evacuation time based on occupant count and inputs that materially affect walking or door flow. It is intended only for planning and training scenarios, not for live operational or real-time incident decision support. Results are estimates and do not replace approved emergency procedures or a certified fire/life-safety analysis.

## Inputs - every value the user enters, with units, ranges, defaults, and the control used to enter it
- Occupancy: user-entered current occupant count for the Marriott Center, from 100 to 2000 whole people, with a default of 500 people. Use a number entry and slider. Values outside the range are clamped to the nearest limit.
- Door availability: preset scenario dropdown controlling whether doors are open. Options are all doors available, exit doors only, and individual selections for the north, south, east, and west groups of doors. Presets update only the named groups and retain other selections. Exit doors means every designated exterior exit door. The default preset is all doors available.
- Obstacles: dropdown for one fixed obstacle scenario: no additional obstacles, rows of seats, aisle restrictions, or stairway/level restrictions. Selecting an obstacle replaces the previous obstacle scenario; there is no severity control. The default obstacle selection is aisle restrictions, and it reduces walking speed and usable aisle capacity while also blocking selected routes to exits. Other obstacle scenarios affect the same walking and route rules according to their type.
- Walking time: derived from the fixed Marriott Center floor plan; no separate walking-time input is provided.
- Occupant placement: occupants are spread randomly across the floor plan. Each occupant moves toward the nearest open exit door that is reachable from its floor-plan position.

## Calculations - the equations, rules, or logic the app uses (formulas, thresholds, lookup tables, how downloaded data is processed), with every variable and its units defined
- Definitions:
  - `Occupancy`, `N`: whole people entering the app, from 100 to 2000 people.
  - `Walking speed`, `v`: 4.3 feet/second before scenario restrictions.
  - `Route distance`, `d`: shortest route distance in feet from the occupant's estimated floor-plan zone to the nearest open exit, using fixed estimated-zone route lengths.
  - `Representative door width`, `w`: 6 feet per exit door.
  - `Door flow rate`, `q`: 24 people/minute/foot of door width.
  - `Open exit count`, `E`: number of open designated exterior exit doors in the selected scenario.
  - `Aisle capacity factor`, `a`: 0.8 for the aisle-restriction scenario.
- Occupant placement: spread `N` occupants randomly across estimated floor-plan zones for each run. The placement varies on repeated runs and is not based on a fixed random seed.
- Routing: remove closed exits and routes blocked by the selected obstacle from consideration. Route each occupant to the nearest reachable open exit.
- Walking speed restrictions:
  - No additional obstacles: walking speed remains `v`.
  - Aisle restrictions: multiply `v` by 0.52. These restrictions reduce usable aisle capacity to 80% and block routes to selected exits.
  - Rows of seats: multiply `v` by 0.50.
  - Stairway/level restrictions: multiply `v` by 0.52.
- Walking time for an occupant is `d / restricted walking speed`, in seconds.
- Door capacity is calculated separately for each open exit: `q * w`, in people/minute. For multiple open exits, sum the capacities: `Total door capacity = q * w * E`, in people/minute.
- Simulate occupants arriving at their nearest open exit over time. Apply a queue at each exit as occupants arrive, with each exit receiving flow at its capacity while occupants remain.
- Time to evacuate is the time in seconds until the final occupant clears the exit route. If no exit doors are open, evacuation time is undefined and the app displays an error.
- Door-flow time is not calculated by a separate per-occupant average. Occupants are processed simultaneously according to arrival time and exit capacity.
- Display the final total evacuation time rounded up to the nearest second.

## Outputs - what the app shows: numbers with units, tables, plots, and diagrams
- `MM:SS` total evacuation time. The time is calculated and updated only when the user presses Calculate.
- Evacuation progress line chart with elapsed time in minutes on the horizontal axis and people evacuated on the vertical axis. Sample and update the cumulative line at one-minute intervals.
- Chart interaction:
  - Update the line live during animation.
  - Include hover tooltips and axis gridlines.
  - Include reset/replay controls.
- Peak evacuation rate: the maximum number of people evacuated during any one-minute interval.
- Numeric summary metrics:
  - Open exit count.
  - Total door capacity in people/minute.
  - Peak evacuation rate in people/minute.
- At the final chart point, show evacuation time and peak evacuation rate. Include occupants remaining as a completion annotation when the result reaches the end.
- Final chart and summary marker behavior: show the chart completion marker when the final occupant clears the exit route.
- Floor-plan animation of the Marriott Center:
  - Represent occupants with dots. The marker count may scale to the screen while every displayed marker represents an individual occupant.
  - Show open exits as labeled markers.
  - Provide Play, Pause, Reset, Speed selector, and Timeline scrubber controls.
  - Offer fixed animation speed options of 1x, 5x, 10x, and 20x.
  - Stop with all occupants removed and show the final chart marker when evacuation completes.
- No-exit state:
  - Show the evacuation time as unavailable and an error beside the floor plan.
  - Keep the animation stopped at the start with all exits visibly closed.
  - Disable animation controls.
- No export/download controls are included.
- Desktop layout: place the chart and animation side by side. Mobile layout: stack the chart and animation vertically.

## Look and feel - the layout, what the app shows when it first opens, and how it works on a phone
- The app opens with 500 occupants, all exits open, and no additional obstacles.
- Use a clean, professional desktop layout with inputs on the left, a detailed illustrated Marriott Center floor plan in the center, and outputs on the right.
- Use raised cards with subtle shadows for the desktop panels. Use balanced compact spacing for engineering-tool readability.
- Use one light neutral accent color plus grey. Reserve additional colors for warnings, errors, and other status feedback. Status must not rely on color alone.
- Use system sans-serif typography and no external font or asset downloads. Display the main numeric evacuation result prominently.
- Input controls are compact and labelled, with number fields and sliders. Labels include concise help text and hover tooltips.
- Place a primary Calculate button below the inputs and a secondary Reset button.
- Reset restores 500 occupants, all exits open, and no additional obstacles.
- The initial floor plan is static, with neutral fills, exit labels, and occupant dots.
- The main result, chart, and floor-plan animation update only after Calculate is pressed.
- While calculating, disable Calculate and briefly show a “Calculating…” state. Dim the previous result until the new result arrives.
- Status feedback uses inline text, icons, borders, and WCAG AA contrast. Success confirmation appears beside the result; errors appear inline beside affected controls.
- Use a high-contrast inset ring for keyboard focus and a subtle elevation change for pointer hover.
- Controls are keyboard accessible and have visible focus indicators.
- Reduced-motion behavior keeps the same controls but reduces occupant-dot movement.
- Mobile layout uses tabs named Inputs, Floor Plan, Chart, and Statistics, in that order. The app opens on the Inputs tab.
- Users can switch tabs by tapping or horizontal swipe.
- On phones, cards become flat and stack within the selected tab. Each card has an optional collapse control.
- Mobile controls use at least 44x44 px tap targets with at least 8 px of spacing.

## Validation - at least one case worked by hand, with exact inputs and expected outputs
- Normal case:
  - Inputs: 500 occupants; all four exit groups (north, south, east, and west) open; four 6-ft representative exit doors; stairway/level restrictions selected.
  - Walking assumption: distribute occupants evenly across the four exit groups, with 125 occupants assigned to each group. Use a fixed 200-ft route for every occupant as a validation-only approximation.
  - Restricted walking speed: 4.3 feet/second x 0.52 = 2.236 feet/second.
  - Walking time: 200 / 2.236 = 89.445 seconds, rounded only at the final result.
  - Total door capacity: 24 people/minute/foot x 6 feet/exit x 4 exits = 576 people/minute.
  - Door clearance time for 500 people at 576 people/minute: 500 / 576 x 60 = 52.083 seconds.
  - Queue model assumption: after occupants arrive, the exits process them simultaneously at the summed capacity.
  - Expected total evacuation time: 89.445 + 52.083 = 141.528 seconds, rounded up to the nearest second = 142 seconds (02:22).
- No-exit case:
  - Inputs: 500 occupants; all exit doors closed.
  - Expected result: evacuation time is undefined, the app displays an error, and the animation remains stopped at the start with all exits visibly closed.
- Boundary cases:
  - 100 occupants with all exits open: valid result and no error.
  - 2000 occupants with all exits open: valid result and no error.
- Invalid-range checks:
  - An entered occupancy below 100 is clamped to 100.
  - An entered occupancy above 2000 is clamped to 2000.
- Rounding check:
  - Use the exact simulated result internally and round only the displayed total up to the nearest second.

## Out of scope - what the app will not do
- Saving or restoring specific user inputs and settings between sessions.
- Live operational or real-time incident-decision support.
- Live sensor data, live occupancy feeds, or integration with building systems.
- Controlling doors, alarms, signage, lighting, or other emergency equipment.
- Replacing approved emergency procedures or a certified fire/life-safety analysis.
- Adapting or converting the fixed Marriott Center floor plan for another venue or user-created venue.
- User editing of the floor plan, exit locations, routes, or representative door width.
- Configurable walking speed or route-distance entry; both remain fixed or derived from the floor plan.
- Door width or walking-speed inputs for individual users or groups.
- Multiple simultaneous obstacle scenarios or obstacle severity controls.
- Custom obstacle types or custom route-blocking rules.
- Scenario comparison, optimization, or recommendations for which scenario is safest.
- Accessibility-specific evacuation cases or accessibility requirements, including mobility assistance and accessibility routes.
- Real-time, live crowd behavior feeds or individualized movement behavior beyond the specified model.
- User-entered or uploaded data files in the app. Floor-plan imports or alternate venue support may be considered in a future version, but are excluded from the current Marriott Center app.
- Exporting or downloading chart images, results, CSV files, or reports.
- Automatic calculation as inputs change; recalculation requires pressing Calculate.
