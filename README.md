# PyBookLid - MacBook Lid Angle Sensor

A Python library for reading MacBook lid angle sensor data on macOS. This library provides real-time access to the built-in lid angle sensor available on modern MacBooks.

## Features

- **One-shot readings** - Get the current lid angle instantly
- **Continuous monitoring** - Stream angle changes in real-time
- **Adaptive audio synthesis** - Generate door creak sounds that respond to movement
- **Context manager support** - Clean resource management
- **Type hints** - Full typing support for better IDE integration
- **Error handling** - Graceful handling of sensor unavailability

## Requirements

- macOS (tested on Apple Silicon)
- Python 3.7+
- Modern MacBook with lid angle sensor (most 2019+ models)

## Installation

Install from PyPI:
```bash
pip install pybooklid
```

Or install with audio features for creak sound generation:
```bash
pip install pybooklid[audio]
```

The library automatically handles the required `DYLD_LIBRARY_PATH` setup for hidapi on macOS.

### Development Installation

1. Clone this repository:
```bash
git clone <repository-url>
cd pybooklid
```

2. Install in development mode:
```bash
pip install -e .
# Or with audio features
pip install -e .[audio]
```

## Quick Start

### One-shot reading
```python
from pybooklid import read_lid_angle

angle = read_lid_angle()
if angle is not None:
    print(f"Current lid angle: {angle:.1f}°")
else:
    print("Sensor not available")
```

### Continuous monitoring
```python
from pybooklid import LidSensor

with LidSensor() as sensor:
    for angle in sensor.monitor(interval=0.1):
        print(f"Angle: {angle:.1f}°")
        if angle < 10:  # Nearly closed
            break
```

### Advanced usage
```python
from pybooklid import LidSensor

# Manual connection management
sensor = LidSensor(auto_connect=False)
try:
    sensor.connect()
    
    # Wait for significant movement
    new_angle = sensor.wait_for_change(threshold=5.0, timeout=10.0)
    if new_angle:
        print(f"Lid moved to {new_angle:.1f}°")
    
    # Monitor with callback
    def on_angle_change(angle):
        print(f"Callback: {angle:.1f}°")
    
    for angle in sensor.monitor(callback=on_angle_change, max_duration=30):
        # Process angle data
        pass
        
finally:
    sensor.disconnect()
```

## API Reference

### `LidSensor` Class

The main sensor interface class.

#### Methods:

- `__init__(auto_connect=True)` - Initialize sensor, optionally auto-connecting
- `connect()` - Connect to the lid angle sensor
- `disconnect()` - Disconnect from the sensor
- `is_connected()` - Check connection status
- `read_angle()` - Get current angle in degrees (0-180°)
- `monitor(interval=0.1, callback=None, max_duration=None)` - Continuous monitoring iterator
- `wait_for_change(threshold=1.0, timeout=None)` - Wait for angle to change

### Convenience Functions

- `read_lid_angle()` - Quick one-shot reading
- `is_sensor_available()` - Check if sensor is available

### Command Line Tools

After installation, you can use the command line tools:

```bash
# Run basic sensor demo
pybooklid-demo

# Run advanced monitoring app
pybooklid-monitor
```

## Sensor Details

The lid angle sensor reports values in degrees:
- **0°** - Lid fully closed
- **90°** - Lid at right angle
- **~180°** - Lid fully open (varies by MacBook model)

The sensor uses Apple's HID interface:
- **Vendor ID**: 0x05AC (Apple)
- **Product ID**: 0x8104 (Sensor Hub)
- **Usage Page**: 0x0020 (Sensor)
- **Usage**: 0x008A (Orientation)

## Error Handling

The library includes comprehensive error handling:

- `LidSensorError` - Base exception for sensor-related errors
- Graceful degradation when sensor is unavailable
- Automatic reconnection attempts
- Safe cleanup on exit

## Compatibility

Tested on:
- MacBook Pro (Apple Silicon)
- macOS Sonoma/Sequoia
- Python 3.12

Should work on most modern MacBooks with lid angle sensors.

## Example Scripts

### Simple Usage (`examples/simple_usage.py`)
Demonstrates basic sensor usage patterns and gesture detection.

### Advanced Monitor (`examples/monitor_app.py`)
Full-featured monitoring application with statistics, logging, and CSV export.

## Troubleshooting

### "Sensor not found" error
- Ensure you're on a supported MacBook model
- Try running with `sudo` if permission issues occur
- Check that no other applications are using the sensor

### Library import errors
- Verify hidapi installation: `pip show hidapi`
- On older macOS, install Homebrew and run `brew install hidapi`

### Inconsistent readings
- The sensor may have noise - use averaging for stable readings
- Some MacBook models may have different angle ranges

## Development

Based on reverse engineering of the IOKit HID interface used by Apple's internal sensor framework.

### Credits

This library was made possible by the reverse engineering work done by Sam Henri Gold in the [LidAngleSensor project](https://github.com/samhenrigold/LidAngleSensor). The key insights about the HID Feature Reports and data format were discovered through that original research.

### Key insights:
- Uses HID Feature Reports (not Input Reports)
- Report ID 1 contains angle data
- Data format: `[report_id, angle_low_byte, angle_high_byte]`
- 16-bit little-endian angle value in degrees

## License

MIT License - see LICENSE file for details.

## Contributing

Contributions welcome! Please test on different MacBook models and report compatibility.

## Support

If this project saved you time, you can support my work via [PayPal](https://paypal.me/dacookingsenpai).
