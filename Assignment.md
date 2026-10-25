## Part 1
*Conditions*
- Reservation is between 1 and 31 nights, inclusive
- Reservation has between 1 and 12 guests, inclusive
- Check in date must be before check out date
- Every night in the requested range is available

*Actions*
- Rase an `UnavailableError`
- Raise a `Valueerror`
- set reservation to `Pending`
- add the $50 cleaning fee
- calculate additional guest charges

## Part 2

***Rule 1*** Check-out date is before Check-in date
***Rule 2*** Number of nights exceeds 31
***Rule 3*** Number of guests exceeds 12
***Rule 4*** Dates entered are unavailable
***Rule 5*** Valid reservation, under 5 or fewer guests
***Rule 6*** Valid reservation, between 6 and 12 guests
| Conditions and Actions         | Rule 1 | Rule 2 | Rule 3 | Rule 4 | Rule 5 | Rule 6 |
| ------------------------------ | :----: | :----: | :----: | :----: | :----: | :----: |
| Dates are valid                |   F    |    -   |    -   |    -   |   T    |   T    |
| Number of nights is valid      |   -    |    F   |    -   |    -   |   T    |   T    |
| Number of guests is valid      |   -    |    -   |    F   |    -   |   T    |   T    |
| Requested nights are available |   -    |    -   |    -   |    F   |   T    |   T    |
| More than five guests          |   -    |    -   |    -   |    -   |   F    |   T    |
| **Create reservation**         |        |        |        |        |   X    |   X    |
| **Add the $50 cleaning fee**   |        |        |        |        |   X    |   X    |
| **Add extra guest fee**        |        |        |        |        |        |   X    |
| **Raise `ValueError`**         |   X    |   X    |   X    |        |        |        |
| **Raise `UnavailableError`**   |        |        |        |    X   |        |        |

## Part 3

### TC01: Invalid dates
 - **Check-In**: October 24
 - **Check-Out**: October 22
 - **Number of guests**: 3
 - **Availability**: all requested nights are availble
 - **Expected Result**: No reservation is created, a `ValueError` is raised
 - **Expected Price**: N/A

 ### TC02: Invalid number of nights
- **Check-In**: October 1
- **Check-Out**: November 2
- **Number of guests**: 3
- **Availability**: All requested nights are available
- **Expected Result**: No reservation is created, a `ValueError` is raised
- **Expected Price**: N/A

### TC03: Invalid number of guests
- **Check-In**: October 24
- **Check-Out**: October 27
- **Number of guests**: 13
- **Availability**: All requested nights are available
- **Expected Result**: No reservation is created, a `ValueError` is raised
- **Expected Price**: N/A

### TC04: Unavailable dates
- **Check-In**: October 24
- **Check-Out**: October 27
- **Number of guests**: 3
- **Availability**: October 25 is unavailable
- **Expected Result**: No reservation is created, an `UnavailableError` is raised
- **Expected Price**: N/A

### TC05: Valid reservation, fewer than 5 guests
- **Check-In**: October 24
- **Check-Out**: October 27
- **Number of guests**: 3
- **Availability**: All requested nights are available
- **Expected Result**: Reservation is created with a `Pending` status
- **Expected Price**: $650
  - 3 nights at $200: $600
  - One-time cleaning fee: $50

### TC06: Valid reservation, between 6 and 12 guests
- **Check-In**: October 24
- **Check-Out**: October 27
- **Number of guests**: 7
- **Availability**: All requested nights are available
- **Expected Result**: Reservation is created with a `Pending` status
- **Expected Price**: $800
  - 3 nights at $200: $600
  - One-time cleaning fee: $50
  - 2 additional guests at $25 per night for 3 nights: $150

## Part 4

### Reservation States

- `Pending`
- `Confirmed`
- `Checked In`
- `Completed`
- `Cancelled`

### Events That Change Reservation States

- Payment succeeds
- Payment fails
- Guest cancels
- Guest checks in
- Guest checks out

## Part 5
| Current State | Event                      | Expected State | Expected Result                                          |
| ------------- | -------------------------- | -------------- | -------------------------------------------------------- |
| Pending       | Payment succeeds           | Confirmed      | The reservation is confirmed.                            |
| Pending       | Payment fails              | Cancelled      | The reservation is cancelled and the dates are released. |
| Pending       | Guest cancels              | Cancelled      | The reservation is cancelled and the dates are released. |
| Confirmed     | Guest checks in            | Checked In     | The guest is checked in.                                 |
| Checked In    | Guest checks out           | Completed      | The reservation is completed.                            |
| Confirmed     | Guest cancels              | Cancelled      | The reservation is cancelled and the dates are released. |
| Cancelled     | Guest attempts to check in | Cancelled      | An `InvalidStateError` is raised.                        |
| Completed     | Guest attempts to cancel   | Completed      | An `InvalidStateError` is raised.                        |
| Checked In    | Guest attempts to cancel   | Checked In     | An `InvalidStateError` is raised.                        |

## Part 6

### State Transition Test Cases

| Test ID | Starting State | Event | Expected State | Expected Result | Transition Type |
|---------|----------------|-------|----------------|-----------------|-----------------|
| ST01    | Pending        | Payment succeeds | Confirmed | Reservation is confirmed | Valid |
| ST02    | Pending        | Payment fails | Cancelled | Reservation is cancelled | Valid |
| ST03    | Pending        | Guest cancels | Cancelled | Reservation is cancelled | Valid |
| ST04    | Confirmed      | Guest checks in on scheduled date | Checked In | Guest is checked in | Valid |
| ST05    | Checked In     | Guest checks out | Completed | Reservation is completed | Valid |
| ST06    | Confirmed      | Guest cancels | Cancelled | Reservation is cancelled | Valid |
| ST07    | Cancelled      | Guest attempts to check in | Cancelled | An `InvalidStateError` is raised | Invalid |
| ST08    | Completed      | Guest attempts to cancel | Completed | An `InvalidStateError` is raised | Invalid |
| ST09    | Checked In     | Guest attempts to cancel | Checked In | An `InvalidStateError` is raised | Invalid |

### ST10: Complete Reservation Sequence

- **Starting State**: Pending
- **Events**:
  1. Payment succeeds → `Confirmed`
  2. Guest checks in on the scheduled date → `Checked In`
  3. Guest checks out after the stay → `Completed`
- **Expected State**: Completed
- **Expected Result**: Reservation successfully transitions through all stages and is marked as `Completed`.
- **Transition Type**: Valid

### ST11: Cancellation Sequence

- **Starting State**: Pending
- **Events**:
  1. Payment succeeds → `Confirmed`
  2. Guest cancels → `Cancelled`
  3. Guest attempts to check in → `InvalidStateError`
- **Expected State**: Cancelled
- **Expected Result**: Reservation is cancelled successfully. Any further attempt to change its state raises an `InvalidStateError`.
- **Transition Type**: Valid sequence followed by an invalid transition