# Scooter Sharing System Description

Our system models a scooter sharing platform, similar to services like Bolt. The architecture is centrally managed by a MobileApp class, which serves as the entry point and aggregates specific controllers—ScooterController, TripController, PaymentController, MapService, and UserController—to orchestrate the interaction between users, the physical fleet, and the backend logic.
Scooter fleet and lifecycle.

## Scooter fleet. 

Every Scooter is identified by scooterId and tracks its batteryLevel, currentLocation, qrCode, and the zoneId of the ParkingZone where it is parked. Each scooter owns exactly one ScooterStatus instance, capturing the last status change (lastUpdated) and the current status chosen from the statusType enumeration with its four literals (available, inUse, charging, outOfService).

The ScooterController manages the active fleet (via the activeScooter association and setStatus operation) and enforces a strict lifecycle defined by the system's State Chart:

    Available: The scooter performs broadcastLiveLocation(). From here, reserveRequested transitions it to ReservationPending (where startReservationTimer() runs), or batteryLowDetected auto-dispatches it to Charging.
    Ride In Progress: Upon a successful unlock (via unlockViaApp or unlockViaKeycard), the scooter enters this composite state. It cycles through internal states:
        Navigating: The main riding state where updateLocation() and incrementActiveTime() occur.
        Paused: Entered if the rider stops temporarily (holdCurrentPosition).
        ParkingCheck: Triggered by riderEndRide, where the system executes validateZoneWithMap() to ensure compliance.
        ConnectionIssue: Handles signal loss via attemptReconnect.
    PostRideProcessing: Once the ride ends validly, the system computes costs and awaits payment confirmation. If paymentAuthorized, the scooter returns to Available (if battery ≥ 30%) or transitions to Charging (if battery < 30%).
    Maintenance: Transitions like technicianPickup move the scooter to OutOfService for diagnosis, while maintenanceComplete returns it to service.

## Trips and payments.

Riders start Trip instances, managed by the TripController (operations startTrip, endTrip). Each trip references exactly one scooter and one rider and is uniquely identified by tripId while storing startLocation, endLocation, duration, travelDistance, and a derived attribute /cost (calculated from duration + travelDistance).

The workflow follows a strict sequence detailed in the Activity Diagram:

    Locate & Unlock: The rider uses the app to locate a scooter. If available, they choose to unlock via App or Keycard.
    Ride: The rider operates the scooter.
    End & Validate: When ending the ride, the system checks if the scooter is "Parked in allowed zone?"
        If No, the Apply parking violation fee logic is triggered.
        If Yes, the flow proceeds to Compute trip cost.
    Financial Settlement: The process continues sequentially: Create trip record → Generate trip invoice → Process payment → Confirm payment → Update scooter status → End.

The PaymentController handles the financial backend (payForTrip, makeInvoice). It generates exactly one PaymentTransaction (transactionId, timestamp, amount, paymentMethod) and a one-to-one Invoice (invoiceNumber, issuedDate, billingDetails). Both artifacts correspond to a single Trip.

## User hierarchy and Recruitment.

All people interact via the abstract User class (userId, name, phone, email), managed by the UserController (addUser, loggedInUser).

Riders:
Rider specializes User and owns exactly one BankAccount (accountId, accountDetails, balance) used to pay for trips. They may optionally link a physical KeyCard (cardId, issuedDate) for quick access.

HR and Recruitment:
The system includes a RecruitmentController to manage hiring logic.

    JobCandidates: Applicants are tracked as JobCandidate objects with resumeText, appliedPosition, and a status (from the CandidateStatus enum: waitingForReview, rejected, withdrawn). Candidates can perform the withdraw() operation.
    Recruitment Process: The controller provides operations to convertToEmployee(candidate), deleteJobCandidate(candidate), and updateCandidateStatus.
    Employees: Once hired, a candidate becomes an Employee (monthlySalary, department).
    Managers: A HumanResourceManager is a specialized Employee who manages other employees. They have a recruitmentQuota and operations to accept(candidate) or reject(candidate).

## Operations staff.

ScooterTechnician (specializing Employee) uses the ScooterController to perform maintenance (maintenanceComplete, technicianDiagnose). They are provided with exactly one Car (humanCapacity, scooterCapacity), which supports between one and four technicians (1..4 multiplicity). Technicians may maintain many scooters, while each scooter is assigned to at most one active technician at a time.
Physical infrastructure.

The MapService contains the defined ParkingZone instances (zoneId, name, location). It provides critical operations for the system's logic:

    getScooterLocation(scooter): Retrieves real-time coordinates.
    locationInParkingZone(): Returns a boolean used during the ParkingCheck state and the "Parked in allowed zone?" decision node in the activity workflow.

## Process overview.

Riders discover a scooter via the MobileApp (backed by MapService), unlock it using the app or an optional key card, and complete a trip. On completion, the MapService validates the parking zone. If valid, the TripController finalizes the cost, and the PaymentController issues the transaction and invoice. Simultaneously, the ScooterController updates the scooter's state from PostRideProcessing back to Available or Charging. Technicians intervene when scooters enter OutOfService or Charging states, physically moving and diagnosing the fleet.
