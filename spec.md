## Stadium Evacuation

## App overview
This is a single-file planning and training web app for the Marriott Center. It accepts occupancy, door availability, and obstacle scenarios, then estimates evacuation timing using a fixed floor-plan model. It calculates total evacuation time plus 50% and 90% evacuation timestamps, displays evacuated and remaining-occupant progress, and animates occupants moving to open exit doors. It is for desktop and mobile planning use, not live incident operations or certified life-safety analysis.

## File placement
- The app code and UI will be written in top-level `index.html`, as required for GitHub Pages.
- Supporting data used to implement fixed routes, door groups, or floor-plan values will go in `01_data/`.
- Hand calculations and test data used to verify the app will go in `02_validation/`.
- Screenshots and exported planning results will go in `03_output/`.
- Notes and sketches used during layout or simulation planning will go in `00_planning/`.
- `spec.md`, `AGENTS.md`, and `prompts.md` remain at the top level. Project implementation should not add unrelated files to the repository root.

## Purpose
A planning and training app for facilities managers, building operators, event staff, emergency responders, and planning personnel. It models the Marriott Center floor plan and estimates total evacuation time, 50% evacuation time, and 90% evacuation time based on occupant count and inputs that materially affect walking or door flow. It is intended only for planning and training scenarios, not for live operational or real-time incident decision support. Results are estimates and do not replace approved emergency procedures or a certified fire/life-safety analysis.

## Inputs - every value the user enters, with units, ranges, defaults, and the control used to enter it
- Occupancy: user-entered current occupant count for the Marriott Center, from 100 to 2000 whole people, with a default of 500 people. Use a number entry and slider. Values outside the range are clamped to the nearest limit.
- Door availability: preset scenario dropdown controlling whether doors are open. Options are all doors available, exit doors only, and individual selections for the north, south, east, and west groups of doors. Presets update only the named groups and retain other selections. Exit doors means every designated exterior exit door. The default preset is all doors available.
- Obstacles: click-open multi-select dropdown for no additional obstacles, rows of seats, aisle restrictions, and stairway/level restrictions. “No additional obstacles” is exclusive and selecting it clears all restrictions. Any combination of the three restriction scenarios may be selected at once, with no severity control. The default selection is no additional obstacles. When combined, each selected restriction applies its walking, aisle, capacity, or route rule concurrently.
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
- Record timestamps when the number of occupants remaining reaches 50% and 10% of the initial occupancy. The 50% timestamp is the time at which at least 50% of occupants have evacuated; the 90% timestamp is the time at which at least 90% of occupants have evacuated.
- Round the displayed 50% and 90% timestamps up to the nearest second.
- Calculate occupants remaining over time as `N - cumulative occupants evacuated`, for each point in the simulation.

## Outputs - what the app shows: numbers with units, tables, plots, and diagrams
- `MM:SS` total evacuation time. The time is calculated and updated only when the user presses Calculate.
- `MM:SS` timestamp for 50% evacuation.
- `MM:SS` timestamp for 90% evacuation.
- Evacuation progress line chart with elapsed time in minutes on the horizontal axis and people evacuated on the vertical axis. Sample and update the cumulative line at one-minute intervals.
- Remaining occupants line chart or line on the same chart with elapsed time in minutes on the horizontal axis and occupants remaining on the vertical axis. Update it live during the animation and sample it at one-minute intervals.
- Clearly distinguish the people-evacuated and people-remaining lines using separate labels and line styles.
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
  - With each occupant's route and door-flow queue simulated consistently across the fixed 200-ft validation routes, the 50% timestamp occurs when 250 occupants are evacuated and the 90% timestamp occurs when 450 occupants are evacuated. Exact timestamps depend on the arrival and queue simulation and must be recorded from the implemented simulation rather than inferred from total time alone.
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
- Obstacle severity controls and custom obstacle types. Combined modeled obstacle scenarios are supported.
- Custom obstacle types or custom route-blocking rules.
- Scenario comparison, optimization, or recommendations for which scenario is safest.
- Accessibility-specific evacuation cases or accessibility requirements, including mobility assistance and accessibility routes.
- Real-time, live crowd behavior feeds or individualized movement behavior beyond the specified model.
- User-entered or uploaded data files in the app. Floor-plan imports or alternate venue support may be considered in a future version, but are excluded from the current Marriott Center app.
- Exporting or downloading chart images, results, CSV files, or reports.
- Automatic calculation as inputs change; recalculation requires pressing Calculate.

## Build stages

### Stage 1 - Page layout mockup
Build the static visual structure in `index.html` before adding behavior.

- Create desktop regions for inputs on the left, the Marriott Center floor plan in the center, and results/chart controls on the right.
- Add compact controls, labels, raised cards, a main result area, chart placeholder, floor-plan placeholder, and mobile tabs.
- Establish the light-neutral and grey palette, typography, spacing, focus, hover, and responsive breakpoints.
- Put the mockup code in top-level `index.html`; put optional layout sketches or notes in `00_planning/`.

Check at completion:
- The default 500-person, all-exits-open, no-obstacle scenario is represented visually.
- Inputs, floor plan, results, and chart are positioned clearly.
- Desktop is balanced and mobile has Inputs, Floor Plan, Chart, and Statistics tabs.
- The page has usable keyboard focus and narrow-screen behavior.

### Stage 2 - Inputs and UI state
Build controls and client-side state.

- Add the occupancy number field and slider for 100-2000 people.
- Add the door preset control and north/south/east/west groups.
- Add the multi-select obstacle dropdown and reset behavior. Allow no additional obstacles or any combination of aisle restrictions, rows of seats, and stairway/level restrictions.
- Add Calculate and loading/error states.
- Keep control markup and behavior in top-level `index.html`.

Check at completion:
- 500 is the default occupancy and all exits are open.
- Values below 100 and above 2000 are clamped.
- Preset changes update only the intended exit groups.
- Reset restores the default scenario.
- No-exit selection shows an inline error and disables animation.

### Stage 3 - Simulation and calculations
Implement the fixed Marriott Center route and evacuation model.

- Use random occupant placement with no fixed seed.
- Apply walking speed, route distance, obstacle, door-flow, queue, and no-exit rules.
- Record total, 50%, and 90% evacuation times.
- Build cumulative evacuated and remaining-occupant time series.
- Put fixed routes, exit groups, door widths, or lookup data in `01_data/` if separate project files are needed.
- Put the implementation code and equations used for the app in top-level `index.html`.
- Put hand-calculation records and test-case inputs/expected results in `02_validation/`.

Check at completion:
- The normal result rounds to 142 seconds (02:22).
- The 50% threshold occurs at 250 evacuated and the 90% threshold at 450 evacuated in the stated test.
- All closed exits return an undefined time and error.
- 100 and 2000 occupants remain valid boundary cases.

### Stage 4 - Charts and floor-plan animation
Build output and visualization behavior.

- Display MM:SS results for total, 50%, and 90% evacuation.
- Add two labelled one-minute series: people evacuated and people remaining.
- Add chart tooltips, gridlines, final markers, and peak evacuation rate.
- Animate occupants, open exit labels, playback controls, and completion behavior.
- Keep chart and animation code in top-level `index.html`.
- Place screenshots or exported chart/result captures in `03_output/` only when generated.

Check at completion:
- The chart updates during playback and distinguishes both series.
- Peak evacuation rate uses one-minute intervals.
- 50%/90% timestamps match simulation threshold events.
- Speed controls work and completion stops with final markers.
- No-exit mode shows closed exits and disables playback controls.

### Stage 5 - Polish, responsive behavior, and verification
Complete cross-device and accessibility checks.

- Apply desktop and mobile panel behavior and mobile tabs.
- Add keyboard focus, visible status feedback, reduced motion, and 44x44 px controls.
- Run normal, no-exit, boundary, clamping, and rounding validation cases.
- Fix only issues revealed by validation and specification review.

Check at completion:
- Desktop and phone layouts match the intended hierarchy.
- Keyboard navigation and status messages do not rely on color alone.
- Reduced-motion playback limits dot movement.
- The app excludes exports, saved settings, live feeds, and alternate floor-plan imports.
- No unrelated files are added to the repository root.
