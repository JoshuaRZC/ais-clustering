# Automatic Identity System Clustering

AIS Transceivers (Transponders) share the radio bandwidth (161.975 MHz and 162.025 MHz) allocated to AIS operation using Time Division Multiple Access (TDMA) techniques. AIS typically operates on two parallel VHF Marine Band radio frequency channels and each channel is shared in time between multiple users by dividing channel access into 2250 ‘slots’ per minute

Each AIS transceiver must determine its own TDMA slot allocation, and critically, it must avoid using a slot that is in use by another vessel within reception range in order to avoid transmissions clashing.

## Transponder Mechanism

### Time Slot Calculation

- Slot Reservation: To simplify the time slot calculation, the AIS Transponders book the slots before sending message. And they use the same timeslot (`TS`) for several messages (`STO`).
- Special Maneuver: When the boat changes the speed or has a special maneuver, it sends message with `id=3`. To adjust the frequency, it needs to send new messages which can have two different patterns. One pattern is to keep the new slot(`keepflag = 1`). The other one is to reserve a new timeslot (`TS_new`xcx).

### Message-Sending Rule

- The frequency of messages depend on the speed of the boat (`sog`) and course changing (`Course`)

|Dynamic Condition|Nominal Reporting Interval |
|---|---|
| Ship at Anchor or Moored, and not moving faster than 3 knots |  3 minutes |
| Ship at Anchor or moored, and moving faster than 3 knots | 10 seconds |
| Ship 0-14 knots | 10 seconds |
| Ship 0-14 knots and changing course | 10/3 seconds |
| Ship 14-23 knots | 6 seconds |
| Ship 14-23 knots and changing course | 2 seconds |
| Ship > 23 knots | 2 seconds |
| Ship > 23 and changing course | 2 seconds |

## Data

### Raw Data

- File-Based Splitting
- Data Field: The −1 in the csv are non documented values (NaN values with numpy)
    - `channelNumber`: One of the two channels.
    - `id`: Different behaviors of the boat. 1: straight line constant speed, 3: acceleration, 5: static information every 6 min (departure, destination, boat identity).
    - `NavStatus`: 0 = under way using engine, 1 = at anchor, 2 = not under command, 3 = restricted maneuverability, 4 = constrained by the drought, 5 = moored... 15 = undefined.
    - `SlotOffset`: When STO = 0 difference between the new slot number and the old one plus 2250.
    - `SlotNumber`: Slot number in AIS message (between 0 and 2249 – a lot of NaN values).
    - `STO`: Initially sampled between 5 and 7 then incrementally decrease until it reached 0, then a new time slot must be chosen.
    - `SlotIncrement`: When acceleration (`id = 3`) need to produce message every 3.3 s. Each channel produces a message every 6.6 s, the slot increment is therefore generally around 6.6·2250 / 60 = 247.
    - `KeepFlag`: Set to TRUE if the slot remains allocated for one additional frame.
    - `TS`: Computed Slot number (almost no `NaN` value).
    - `Lat`: Latitude.
    - `Lon`: Longitude.
    - `Sog`: Speed.
    - `thresholdSog`: Different discrete value depending on the speed (if Sog∈ [0, 3]: 0, if Sog= 3: 1, if Sog∈ (3, 14): 2, if Sog= 14: 3, if Sog∈ (14, 23): 4, if Sog= 23: 5, if Sog> 23: 6...)
    - `Course`: Direction (different with Heading, if there is some sea current).
    - `Heading`: Heading.
    - `changeHeading`: Change of Heading.
    - `RepeatIndicator`: Some rare messages sent when a boat transmits the message of another boat.
    - `SpecialManoeuvre`: 0 = not available = default 1 = not engaged in special maneuver, 2 = engaged in special maneuver (i.e. regional passing arrangement on Inland Waterway).
    - `AISVersion`: Version of AIS used.
    - `toa`: Time of emission.
    - `DiffToa`: Difference in time with previous emission.

### Feature Data

- Feature Data: The feature variables used to cluster the data is related to Constructor Signatures.
- Feature Data Constructor Signature: Constructor Signature is the small difference between information sent by each boat.
    - Synchronization errors (Difference between `TS` and `SlotNumber`)
    - Regularity of messages depending on the speed/acceleration (Speed and Frequency of Message)
    <!-- - Emissions characteristics during special maneuvers (The `SlotIncrement`, and `KeepFlag` of `NavStatus=3` and `id=3`) -->
    - Sampling of STOs and Slot numbers (The range of `STO` and New Slot Number, Distribution)
    - Reservation of a Timeslot before sending a message with `id =  5` (Investigate `id = 3`).
    - Alternation between Channels (Normally it should always alternate, but it is possible that some boats do not respect it ).
- Feature Data Candidate:
    - `channelNumber`: One of the two channels.
    - `id`: Different behaviors of the boat. 1: straight line constant speed, 3: acceleration, 5: static information every 3 min (departure, destination, boat identity).
    - `NavStatus`: 0 = under way using engine, 1 = at anchor, 2 = not under command, 3 = restricted maneuverability, 4 = constrained by the drought, 5 = moored... 15 = undefined.
    - `SlotOffset`: When STO = 0 difference between the new slot number and the old one plus 2250.
    - `SlotNumber`: Slot number in AIS message (between 0 and 2249 – a lot of NaN values).
    - `STO`: Initially sampled between 5 and 7 then incrementally decrease until it reached 0, then a new time slot must be chosen.
    - `SlotIncrement`: When acceleration (`id = 3`) need to produce message every 3.3 s. Each channel produces a message every 6.6 s, the slot increment is therefore generally around 6.6·2250 / 60 = 247.
    - `KeepFlag`: Set to TRUE if the slot remains allocated for one additional frame.
    - `TS`: Computed Slot number (almost no `NaN` value).
    - `Sog`: Speed.
    - `thresholdSog`: Different discrete value depending on the speed (if Sog ∈ [0, 3]: 0, if Sog = 3: 1, if Sog ∈ (3, 14): 2, if Sog = 14: 3, if Sog ∈ (14, 23): 4, if Sog = 23: 5, if Sog > 23: 6...)
    - `Course`: Direction (different with Heading, if there is some sea current).
    - `Heading`: Heading.
    - `changeHeading`: Change of Heading.
    - `SpecialManoeuvre`: 0 = not available = default 1 = not engaged in special maneuver, 2 = engaged in special maneuver (i.e. regional passing arrangement on Inland Waterway).
    - `toa`: Time of emission.
    - `DiffToa`: Difference in time with previous emission.
