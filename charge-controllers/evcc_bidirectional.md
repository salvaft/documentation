> [!UPDATE] {docsify-updated}
# EVCC Bidirectionality

## Introduction

The second version of ADVANTICS generic interface for EVCC extends the available CAN bus based API by introducing bidirectionality to the controller.

With the support of ISO 15118-20, bidirectional power transfer became possible through 2 control modes defined by the protocol.
In Dynamic control mode, besides the static power transfer parameters, vehicle provides the Energy requests and the range for bidirectional cycling (V2X energy requests).

It is possible to specify the absolute energy requests including the range for bidirectional cycling (V2X energy requests) within the battery capacity limit. Based on these parameters and the state of charge of the battery we will then dynamically calculate relative energy requests and V2X cycling range limits and forward them to the charger.

In addition, the vehicle will be able to specify the time of departure which can be used in dynamic mode.

## Relevant config entries

### type

In the `[vehicle]` section. It is an existing config entry with a new valid value.

<figcaption>Example</figcaption>

    type = Advantics_Generic_v2

This entry specify which vehicle interface type is to be used. For the Generic Interface supporting
bidirectional power transfers, the valid values are:

> [!DLIST|label:Advantics_Generic_v2]
> The second version of the generic interface

### is_bidirectional

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    is_bidirectional = true

To allow bidirectionality, set to `true`.

Default to false.

### energy_capacity

In the `[vehicle]` section. It is an extisting entry that was optional. For bidirectionality it is
required.

<figcaption>Example</figcaption>

    energy_capacity = 33000

Energy capacity of the battery, in Wh. Required for bidirectional power transfers. Otherwise it is
optional but recommended. Used for various informational calculations (SoC, remaining time, etc.).

Default to 0 Wh.

### max_discharge_current

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    max_discharge_current = 30

Maximum allowed current in discharge phase.

Default to 0 A.

### min_discharge_power

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    min_discharge_power = 2000

Minimum discharge power, in Watts.

Default to 0 W.

### max_discharge_power

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    max_discharge_power = 20000

Maximum discharge power, in Watts.

Default to 0 W.

### min_energy_request

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    min_energy_request = 0

Absolute minimum energy request, in Wh. Can be dynamically overloaded through the Generic CAN API.

Default to 0 Wh.

### max_energy_request

In the `[vehicle]` section. It is a new config entry.

<figcaption>Example</figcaption>

    max_energy_request = 33000

Absolute maximum energy request, in Wh. Can be dynamically overloaded through the Generic CAN API.

Default to 0 Wh.

### enable_iso_part20

In the `[ccs]` section. It is a new config entry.

<figcaption>Example</figcaption>

    enable_iso_part20 = true

Loads support for ISO 15118-20.

Default to true.

> [!NOTE]
> When enabled, the `ccs-evcc` application is much longer to load. This performance point will be
> addressed later.

### iso_part20_dc_priority

In the `[ccs]` section. It is a new config entry.

<figcaption>Example</figcaption>

    iso_part20_dc_priority = 1

Sets the priority used in AppProtocol for ISO 15118-20 DC schema.

Default to 1.

> [!NOTE]
> Other priority entries are:
>
> - `iso_part2_priority` (default to 2)
> - `iso_ed1_priority` (deprecated name for `iso_part2_priority`, default to 2)
> - `din_priority` (default to 3)

## Changes in the EVCC Generic CAN interface v2:

<div class="compact-table">

| Name | ID | Length | Direction | Cycle time | Difference |
|------|----|--------|-----------|------------| ----------- |
| [EV_Information](#EV_Information) | 0x610 | 3 | IN | 100 | New signal |
| [EV_Energy_Request](#EV_Energy_Request) | 0x614 | 6 | IN | 100 | New message |
| [EV_V2X_Energy_Request](#EV_V2X_Energy_Request) | 0x615 | 4 | IN | 100 | New message |
| [EV_Extra_BPT_Information](#EV_Extra_BPT_Information) | 0x616 | 4 | IN | Optional | New message |

</div>

## EV_Information

<div class="noheader-table small-table compact-table">

| * | * |
|---|---|
| **Frame ID** | 0x610 |
| **Length [Bytes]** | 3 |
| **Periodicity [ms]** | 100 |
| **Direction** | IN |

</div>

### Description

Information provided by the vehicle.


### Payload


<div class="small-table compact-table">

| Signal | Length (bits) | Type |
|--------|---------------|------|
| State_of_Charge | 8 | Unsigned |
| Energy_Capacity | 16 | Unsigned |

</div>


### Payload description

#### State_of_Charge :id=EV_Information-State_of_Charge

Battery SoC in percent (only used in HLC mode).


<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 0 | 8 | Unsigned | % | 1 | 0 | 0 | 100 |

</div>

#### Energy_Capacity :id=EV_Information-Energy_Capacity

The energy capacity of the EV battery.
In case this value is 0 we default to the config file entry `energy_capacity`.

Pleaase note that providing the energy capacity of the battery is mandatory if ISO15118-20 is used.


<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 8 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

## EV_Energy_Request

<div class="noheader-table small-table compact-table">

| * | * |
|---|---|
| **Frame ID** | 0x614 |
| **Length [Bytes]** | 6 |
| **Periodicity [ms]** | 100 |
| **Direction** | IN |

</div>

### Description

ISO 15118-20 specific message.
The energy of the EV corresponding to the SOC specified by the owner.
The energy request is represented by an energy range including a target energy request.

Energy requests should satisfy the following relationship:
```
Minimum_Energy_Request ≤ Target_Energy_Request ≤ Maximum_Energy_Request
```

### Payload


<div class="small-table compact-table">

| Signal | Length (bits) | Type |
|--------|---------------|------|
| Target_Energy_Request | 16 | Unsigned |
| Minimum_Energy_Request | 16 | Unsigned |
| Maximum_Energy_Request | 16 | Unsigned |

</div>


### Payload description

#### Target_Energy_Request :id=EV_Energy_Request-Target_Energy_Request

The energy of the EV corresponding to the target SOC.
The target energy request can be lower than the current energy level present in the battery represented by the SoC.
This represents a discharge request. More details available in the ISO15118-20 documentation.

<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 0 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

#### Minimum_Energy_Request :id=EV_Energy_Request-Minimum_Energy_Request

The energy of the EV corresponding to the minimum SOC.
In case this message is not sent we default to the config file entry `min_energy_request`.


<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 16 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

#### Maximum_Energy_Request :id=EV_Energy_Request-Maximum_Energy_Request

The energy of the EV corresponding the to maximum SOC.
In case this message is not sent we default to the config file entry `max_energy_request`.
This value will be capped by Energy_Capacity.


<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 16 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

## EV_V2X_Energy_Request

<div class="noheader-table small-table compact-table">

| * | * |
|---|---|
| **Frame ID** | 0x615 |
| **Length [Bytes]** | 4 |
| **Periodicity [ms]** | 100 |
| **Direction** | IN |

</div>

### Description

ISO 15118-20 specific message. (Optional)
Indicates a preferred operational V2X energy range for bidirectional cycling.
V2X energy requests should satisfy the following relationships:
```
Minimum_Energy_Request ≤ Minimum_V2X_Energy_Request
Maximum_V2X_Energy_Request ≤ Maximum_Energy_Request
```

### Payload


<div class="small-table compact-table">

| Signal | Length (bits) | Type |
|--------|---------------|------|
| Minimum_V2X_Energy_Request | 16 | Unsigned |
| Maximum_V2X_Energy_Request | 16 | Unsigned |

</div>


### Payload description

#### Minimum_V2X_Energy_Request :id=EV_Energy_Request-Minimum_V2X_Energy_Request

The minimum energy level for the bidirectional cycling activity range.

<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 0 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

#### Maximum_Energy_Request :id=EV_Energy_Request-Maximum_Energy_Request

The maximum energy level for the bidirectional cycling activity range.

<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 16 | 16 | Unsigned | kWh | 0.01 | 0 | 0 | 655 |

</div>

## EV_Extra_BPT_Information

<div class="noheader-table small-table compact-table">

| * | * |
|---|---|
| **Frame ID** | 0x616 |
| **Length [Bytes]** | 4 |
| **Periodicity [ms]** | 100 |
| **Direction** | IN |

</div>

### Description

ISO 15118-20 specific message.
EV extra bidirectional power transfer information.
### Payload


<div class="small-table compact-table">

| Signal | Length (bits) | Type |
|--------|---------------|------|
| Departure_Time | 32 | Unsigned |

</div>


### Payload description

#### Departure_Time :id=EV_Extra_BPT_Information-Departure_Time

Indicate when the EV intends to finish the charging process.
Represents the offset in seconds from the point in time of sending this message.
<div class="small-table compact-table">

| Start bit | Length (bits) | Type | Unit | Scale | Offset | Min | Max |
|-----------|---------------|------|------|-------|--------|-----|-----|
| 0 | 32 | Unsigned | Seconds | 1 | 0 | 0 | 4294967296 |

</div>