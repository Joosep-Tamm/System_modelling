# How can environmental aspects be integrated into Tasks #2-4?

## Task #2: State Chart (Handling Physical & Network Constraints)

In a deployed environment, a mobile app and scooter communicate over imperfect networks (4G/5G) and interact with physical sensors. You could model how the system reacts to signal loss or hardware limits.

    Composite State (Connection Handling):
        Create a composite state called OnlineOperation (which includes your standard states like Available, Reservation, RideInProgress).
        The Environmental Event: signalLost.
        The Transition: Transitions out of OnlineOperation to a state called OfflineMode.
        History Pseudo-state (H): When the environment changes (event: signalRestored), the system transitions back to the History state inside OnlineOperation. This ensures that if the rider was in Navigating when the signal dropped (e.g., entering a tunnel), the app remembers exactly where they were rather than resetting the ride.

    Guard Conditions (Physical Limits):
        Temperature: Add a transition from Available to OutOfService triggered by checkDiagnostics().
        Guard: [internalTemp > 45°C OR internalTemp < -10°C]. This models the physical environment (weather) preventing the scooter from operating safely.

## Task #3: Activity Diagram (Deployment Topology & External Systems)

Activity diagrams are excellent for showing the boundaries between the different parts of the deployed environment. You could use partitions to represent these distinct environments.

    Partitions: Divide the diagram into 3 vertical columns:
        Rider (Physical World): Actions the human does (e.g., "Scans QR Code", "Physically Parks Scooter").
        Mobile App (Client Device): Actions happening on the phone (e.g., "Request Unlock", "Display Map").
        Backend System (Cloud Server): Actions happening remotely (e.g., "Validate Payment", "Store Trip Data").

    External Service Integration:
        When the "End Ride" action occurs, draw an action node called Request GPS Fix.
        Connect this to an Input Pin or Object Node labeled <<External System>> GPS Satellites. This explicitly models the dependence on the geospatial environment.
        Add a decision node for Network Latency: After "Process Payment," add a decision "Server Response Received?". If No (Time-out), loop back to "Retry" or go to a "Store locally" action. This models the unreliability of the network environment.

## Task #4: Application Class Model (Boundary Classes & Sensors)

To model the deployed environment, you could represent the interfaces between your software and the physical world (the hardware).

    Boundary/Interface Classes:
        Instead of just a generic Scooter class, add classes that represent the hardware drivers.
        GPSSensor: Attributes: latitude, longitude, accuracy. Operations: getCoordinates().
        NetworkAdapter: Operations: connect(), disconnect(), getSignalStrength().
        BatteryManager: Attributes: voltage, temperature, cycles.

    Relationships:
        The Scooter class should have a Composition relationship (black diamond) with these sensor classes (e.g., A Scooter is composed of 1 GPSSensor and 1 BatteryManager). This indicates that the software object relies on these hardware components to exist in the real world.

    Description Sentence (for the assignment):
        "Unlike the model in Assignment #1, this updated model introduces Boundary classes (like GPSSensor and NetworkAdapter) and Swimlanes to explicitly represent the deployment constraints, hardware interfaces, and network topology required for the application to function in a real-world physical environment."
