# Scooter Sharing System Description

Our system models a **scooter** sharing platform, similar to services like Bolt. The platform allows **riders** to locate, unlock, and ride scooters using a mobile application.

## Scooter fleet. 
Every `Scooter` is identified by `scooterId` and tracks its `batteryLevel`, `currentLocation`, `qrCode`, and the `zoneId` of the `ParkingZone` where it is parked (if any). Each scooter owns exactly one `ScooterStatus` instance at a time, and each status record belongs to exactly one scooter, capturing the last status change (`lastUpdated`) and the current `status` chosen from the `statusType` enumeration with its four literals (`available`, `inUse`, `charging`, `outOfService`). 

Use-Case Integration

- Locate scooter (Rider + Technician) uses currentLocation and statusType.
- Update scooter status (Technician) modifies the active ScooterStatus.
- Unlock scooter transitions status to inUse.
- End scooter ride transitions scooters back to available or charging.

Scooter Technicians maintain scooters and update their status through their operational use cases (Maintain scooter, Update scooter status), which map directly to modifying Scooter, ScooterStatus, and associations with technicians.

## Trips and payments.
Riders start `Trip` instances through use cases
- Unlock via mobile app
- Unlock via keycard

Each trip references exactly one scooter and one rider (multiplicity `1` on both associations) and is uniquely identified by `tripId` while storing `startLocation`, `endLocation`, `duration`, `travelDistance`, and computed `cost` (`duration + travelDistance`). Scooters and riders, in turn, participate in many trips over time (multiplicity `0..*`). When a trip finishes, the platform issues exactly one `PaymentTransaction` (with `transactionId`, `timestamp`, `amount`, `paymentMethod`) and a one-to-one `Invoice` (`invoiceNumber`, `issuedDate`, `billingDetails`). Every trip is covered by exactly one invoice and exactly one payment transaction, and each of those artefacts points back to a single trip (bidirectional `1..1` associations). The transaction covers the associated invoice and records the identifiers of the trip and paying user, ensuring auditable billing. Each transaction belongs to exactly one user, while users may accumulate many transactions over time.

When a trip ends (via End a scooter ride):
- The system validates the parking location.
- If parked outside any allowed ParkingZone, the Apply a parking violation fee (extend) use case is triggered.
- Cost is finalized.

After this, Pay for the ride is executed.

When a rider pays:
- The system includes the Generate invoice use case.
- The external Payment Service performs the Process payment use case.

## User hierarchy.
All people interacting with the system inherit from the abstract `User` class (`userId`, `name`, `phone`, `email`). `Rider` and `Employee` specialize this base type:

- A `Rider` owns exactly one `BankAccount` (`accountId`, `accountDetails`) used to pay for trips, and each bank account is linked to exactly one user. Riders may optionally link a physical `KeyCard` (`cardId`, `issuedDate`)—multiplicity `0..1` on the rider side and `1` on the key card side—for quick scooter access. Riders can take many trips.

Riders can:
- Locate scooter
- View parking zones
- Unlock scooter (via app/keycard)
- Start a scooter ride
- End a scooter ride
- Pay for the ride

An `Employee` stores `monthlySalary` and `department`. Every employee is managed by exactly one `HumanResourceManager`, and each manager supervises one or more employees. `Employee` acts as a generalization of the `Human Resource Manager` and `Scooter Technician` classes.

## Operations staff.
`HumanResourceManager` employees oversee the broader workforce, including `ScooterTechnician` employees. Managers are linked to one or more `AdministrativeResponsibility` entries, and each responsibility can itself reference one or more managers, forming a many-to-many relationship that documents shared oversight areas. `ScooterTechnician` specializes `Employee`, maintains scooters, and is provided with exactly one `Car` (with `humanCapacity` and `scooterCapacity`). Each car can support between one and four technicians (`1..4` multiplicity on the car-to-technician association). Technicians may maintain many scooters, while each scooter is assigned to at most one active technician at a time.

HR use cases:
- Add New Employee
- Remove Employee

Scooter Technicians perform:
- Maintain scooter
- Locate scooter
- Update scooter status

## Physical infrastructure.
`ParkingZone` instances (`zoneId`, `name`, `location`) designate legal parking spots. A zone can hold many scooters (`0..*`), while each scooter can belong to at most one zone at a time (`0..1`). When a rider finishes a trip, the application enforces parking inside an allowed zone before charging and invoicing. The external `Map` component (`apiKey`) contains the defined parking zones with multiplicity `1..*` and renders them to customers.

Use cases related to infrastructure:
- View parking zones
- End scooter ride (zone validation)
- Apply a parking violation fee (condition: “parked outside permitted zone”)

## Process overview.
Riders discover a scooter via the mobile app (backed by the `Map` integration), unlock it using the app or an optional key card, and complete a trip. On completion they park in a permitted zone, the system finalizes the trip, updates the scooter’s status to `available` or `charging`, and issues invoicing documentation. Scooter technicians use company cars to rebalance or repair scooters, updating statuses as they go, while HR managers track responsibilities and supervise employees.
