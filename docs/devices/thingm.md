![BusyLight Project Logo][1]

## ThingM blink(1)

[ThingM][0] is a California company that describes themselves as a "device
studio" and offer many different LED based devices; the consumer
oriented blink(1) product line and for prototyping and hacking they
offer the BlinkM line of products. I developed with the blink(1) mk3
device.

### Physical Description

#### blink(1)

The ThingM blink(1) is the smallest USB connected LED light I have
worked with so far. The device mostly consists of a USB-A male
connector with a wraparound transluscent diffusor. The device
dimensions are _roughly_ 1.5 inches long, (excluding USB-A male
connector), 1.25 inches wide and 0.25 inches deep.  The device has two
LEDs; one firing up and the other firing down. My ThingM blink(1) mk3
shipped with an approximately 3 foot long USB cable with a USB-A male
connector and a USB-A female connector.

### Basic Human Interface Device Info

- Vendor ID values:
- I/O Interface: `HID` send_feature_report
- Command Length: 8-bytes

### Command Formats

The Blink family of devices has (at the time of writing) three
different varieties: mark I, mark II and mark III. In general, most
commands are eight bytes long. The first byte is a `report` value
which is either `0x01` or `0x02`. The second byte is an `action` value
which is the ordinal value of an a single character string. The
`action` value determines the format of the rest of the command
buffer.

**Generic Blink Command**
```C
typedef struct {
	unsigned int report; /* 56:63 */
	unsigned int action; /* 48:55 */
	unsigned int arg0;	 /* 40:47 */
	unsigned int arg1;	 /* 32:39 */
	unsigned int arg2;	 /* 24:31 */
	unsigned int arg3;	 /* 16:23 */
	unsigned int arg4;   /* 08:17 */
	unsigned int arg5;   /* 00:07 */
} blink_command_t;
```

#### Color Commands

```C
typedef struct {
	unsigned int report;  /* 56:63 */
	unsigned int action;  /* 48:55 */
	unsigned int red;	  /* 40:47 */
	unsigned int green;	  /* 32:39 */
	unsigned int blue;	  /* 24:31 */
	unsigned int fade_ms; /* 08:23 */
	unsigned int led;     /* 00:07 */
} blink_color_command_t;
```

#### Pattern Commands
```C
typedef struct {
	unsigned int report;  /* 56:63 */
	unsigned int action;  /* 48:55 */
	unsigned int red;	  /* 40:47 */
	unsigned int green;	  /* 32:39 */
	unsigned int blue;	  /* 24:31 */
	unsigned int fade_ms; /* 08:23 */
	unsigned int led;     /* 00:07 */
} blink_pattern_command_t;
```

### Device Operation

#### Activating with a RGB Color

#### Turning the Light Off

### Observations

The device will briefly flash a white color on both LEDs when first
plugged in.

### Functionality Wishlist

[0]: https://thingm.com/products
[1]: https://github.com/JnyJny/busylight/blob/master/docs/assets/BusyLightLogo.png
[H]: https://github.com/libusb/hidapi
