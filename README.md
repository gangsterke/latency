# FB_EventTimer - Latency Measurement Function Block

A flexible Siemens SCL (Structured Control Language) function block for measuring elapsed time between two events in PLC applications. This function block is designed for latency measurement, performance monitoring, and timing analysis in industrial automation systems.

## Features

- **Flexible Trigger Inputs**: Support for both boolean and integer/real signal types
  - Boolean triggers: Rising edge detection on boolean signals
  - Integer/Real triggers: Change detection from 0 to any value (or vice versa)
- **Circular Buffer Logging**: Stores the last 100 measurements with timestamps
- **Real-time Statistics**: Calculates running minimum, maximum, and average values
- **State Machine Architecture**: Robust timing measurement with proper state management
- **Reset Functionality**: Clear all logged measurements with a single reset input

## Inputs

| Input | Type | Description |
|-------|------|-------------|
| `bstart` | Bool | Boolean start trigger (rising edge detection) |
| `bstop` | Bool | Boolean stop trigger (rising edge detection) |
| `rstartstop` | Int | Integer/Real signal: starts timer when value changes from 0 to non-zero, stops when value changes from non-zero to 0 |
| `reset` | Bool | Reset input to clear all log entries (rising edge) |

### Input Usage Notes

- **Mutually Exclusive Inputs**: Only one trigger type can be used at a time
  - If `bstart` is used, `rstartstop` is ignored for start trigger
  - If `bstop` is used, `rstartstop` is ignored for stop trigger
  - If `rstartstop` is used, it handles both start and stop events
- **Supported Combinations**:
  - Boolean + Boolean: `bstart` + `bstop`
  - Boolean + Real: `bstart` + `rstartstop` (or `bstop` + `rstartstop`)
  - Real only: `rstartstop` handles both start and stop

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `elapsedTime_ms` | DInt | Elapsed time in milliseconds (current measurement or running value) |
| `measurementValid` | Bool | Indicates if the current measurement is valid |
| `logCount` | Int | Number of measurements currently in the log (0-100) |
| `minTime_ms` | DInt | Minimum elapsed time from all logged values |
| `maxTime_ms` | DInt | Maximum elapsed time from all logged values |
| `avgTime_ms` | DInt | Average elapsed time from all logged values |

## How It Works

1. **Start Event Detection**: 
   - Boolean mode: Detects rising edge on `bstart`
   - Real mode: Detects change from 0 to non-zero on `rstartstop`
   - Timer starts when start event is detected

2. **Stop Event Detection**:
   - Boolean mode: Detects rising edge on `bstop`
   - Real mode: Detects change from non-zero to 0 on `rstartstop`
   - Timer stops and measurement is logged

3. **Logging**: Each measurement is stored in a circular buffer with:
   - Event ID (trigger value)
   - Elapsed time in milliseconds
   - Timestamp (DTL format)

4. **Statistics**: After each measurement, min, max, and average are recalculated from all valid log entries

## Usage Example

### Example 1: Boolean Triggers

```scl
VAR
    myTimer : FB_EventTimer;
    startSignal : Bool;
    stopSignal : Bool;
END_VAR

// Call the function block
myTimer(
    bstart := startSignal,
    bstop := stopSignal,
    rstartstop := 0,
    reset := FALSE
);

// Access results
IF myTimer.measurementValid THEN
    // Use myTimer.elapsedTime_ms, myTimer.minTime_ms, etc.
END_IF;
```

### Example 2: Real/Integer Signal

```scl
VAR
    myTimer : FB_EventTimer;
    controlSignal : Int;  // 0 = idle, non-zero = active
END_VAR

// Call the function block
myTimer(
    bstart := FALSE,
    bstop := FALSE,
    rstartstop := controlSignal,
    reset := FALSE
);

// Timer starts when controlSignal goes from 0 to any value
// Timer stops when controlSignal returns to 0
```

## Technical Details

- **Language**: Siemens SCL (Structured Control Language)
- **Target Platform**: Siemens S7-1500 / TIA Portal
- **Optimized Access**: Enabled for better performance
- **Log Buffer Size**: 100 measurements (circular buffer)
- **Time Resolution**: Milliseconds (DInt)
- **State Machine**: 2 states (IDLE, TIMING)

## File Structure

```
latency/
├── FB_EventTimer.scl          # Main function block
├── latency test block/         # Test project v1
├── latency test block v2/      # Test project v2
├── latency test block v3/      # Test project v3
└── latencyCheck nodeset.xml    # OPC UA nodeset (if applicable)
```

## Requirements

- Siemens TIA Portal (V15 or later recommended)
- Compatible with S7-1500 series PLCs
- SCL programming language support

## Version History

- **v0.1**: Initial release
  - Basic timing measurement functionality
  - Circular buffer logging (100 entries)
  - Flexible input support (boolean and integer/real)
  - Statistics calculation (min, max, average)

## License

[Specify your license here]

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Author

[Your name/contact information]

## Acknowledgments

This function block was developed for latency measurement and performance monitoring in industrial automation applications.
