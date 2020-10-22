![BusyLight Project Logo][1]

## Kuando BusyLight

The [Kuando BusyLight][0] line of products consists of the Busylight UC Alpha and the
BusyLight US Omega. For development, I tested with the Omega model. I have no
reason to believe that the Alpha varies in any significant way with respect to
it's software configuration.

### Physical Description

#### UC Omega

The UC Omega is vaguely cylindrical in shape, with a slightly wider
base and a narrower top. The device has a permamently attached USB
cord with a USB-A male connector. There is a small side-firing speaker
grill on the side opposite of the cable. The entire device is roughly
1.5 inches in diameter and 1.75 inches tall. The diffuser is secured
to the base, possibly glued, and I was not able to determine how many
LED lights the device has. 

### Basic Human Interface Device Info

- Vendor Id values: 0x27bb
- I/O Interface: HID `write`

### Command Format

The Kuando UC Omega is a [USB HID][H] accessible device whose
attributes are controlled by writing a 64 byte packet to the device.
The 64-byte packet is subdivided by 7 8-byte "step" segements and a
final 8-byte segment containing house-keeping values and a terminating
16-bit [checksum][CHKSUM] field.

The following is a C language structure type definition using bit
fields to illustrate the gross structure of the 64-byte packet.

```C
typedef struct {
	unsigned int step0:       64; /* 448:511 Step command 0 */
	unsigned int step1:       64; /* 384:447 Step command 1 */
	unsigned int step2:       64; /* 320:383 Step command 2 */
	unsigned int step3:       64; /* 256:319 Step command 3 */
	unsigned int step4:       64; /* 192:255 Step command 4 */
	unsigned int step5:       64; /* 128:191 Step command 5 */
	unsigned int step6:       64; /* 064:127 Step command 6 */
	unsigned int sensitivity:  8; /* 056:063 Kuando Box sensitivity 0=hi, 31=low [0,31] */
	unsigned int timeout:      8; /* 048:055 Kuando Box timeout in seconds [1,30] */
	unsigned int trigger;      8; /* 040:047 Kuando Box trigger time in ms [1, 250] */
	unsigned int pad0;        24; /* 016:039 Three byte pad, required to be 0xffffff */	
	unsigned int checksum;    16; /* 000:015 Checksum of 64-byte packet */
} busylight_buf_t;
```

Compared to the [Embrava Blynclight][Embrava] this device is somewhat more
sophisticated in operation. In practice, a single step is sufficient
to carry out simple operations and additional steps allow for more
complicated effects. Of signifcant note, the device **requires** a regular
'keep alive' write, otherwise the device stops executing the current
program and quiesces (turns off the light, stops ringtone).

The `sensitivity`, `timeout` and `trigger` fields are not used by either
the UC Alpha or UC Omega products.

The `checksum` field is computed by treating the 64-byte packet as a
byte array and accumulating the sum of the first 62 bytes into the
last two bytes of the packet.

```C
busylight_buf_t buf;
unsigned char *bytes = &buf;

/* configure steps in buf */

buf.pad0 = 0xffffff;

buf.checksum = 0

for(int i=0; i < 62; i++)
   buf.checksum += bytes[i]
```


#### Step Format

Each 8-byte step has the following general format with four
variations; keep alive, boot loader, reset, and jump.

```C

#define KEEP_ALIVE_STEP       (1 << 3)   /* 0x8 */
#define START_BOOTLOADER_STEP (1 << 2)   /* 0x4 */
#define RESET_DEVICE_STEP     (1 << 1)   /* 0x2 */
#define JUMP_TO_STEP          (1 << 0)   /* 0x1 */

typedef struct {
	unsigned int cmd_hi:  4;  /* 60:63 Step type [Keep alive|Boot|Reset|Jump] */
	unsigned int cmd_lo:  4;  /* 56:59 Keep alive timeout or jump target*/
	unsigned int body:   56:  /* 00:55 */
} busylight_step_t;
```

##### Keep Alive Step Format

```C
typedef struct {
    unsigned int keepalive: 4; /* 60:63 KEEP_ALIVE_STEP */
	unsigned int timeout:   4; /* 56:59 In seconds, [0, 15] */
	unsigned int pad0:     56; /* 00:55 zeros */
} busylight_keepalive_step_t;	
```

##### Boot Loader Step Format

```C
typedef struct {
    unsigned int boot:      4; /* 60:63 BOOTLOADER_STEP */
	unsigned int pad0:     60; /* 00:59 zeros */
} busylight_bootloader_step_t;
```

##### Reset Step Format
```C
typedef struct {
    unsigned int reset:     4; /* 60:63 RESET_DEVICE_STEP */
	unsigned int pad0:     60; /* 00:59 zeros */
} busylight_reset_step_t;
```

##### Jump Step Format
```C
typedef struct {
    unsigned int jump:     4;  /* 60:63 JUMP_TO_STEP */
	unsigned int pad0      1;  /* 59:59 zero */
	unsigned int target:   3;  /* 56:58 Jump target, [0,7] */
	unsigned int repeat:   8;  /* 48:55 Execute this step N times [1,255] */
	unsigned int red:      8;  /* 40:47 8-bit PWM on time [0,100] */
	unsigned int blue:     8;  /* 32:39 8-bit PWM on time [0,100] */
	unsigned int green:    8;  /* 24:31 8-bit PWM on time [0,100] */
	unsigned int on_time:  8;  /* 16:23 time in 0.1 sec steps [0,255] */
	unsigned int off_time: 8;  /* 08:15 time in 0.1 sec steps [0,255] */
	unsigned int update:   1;  /* 07:07 if clear, following audio is ignored*/
	unsigned int ringtone: 4;  /* 03:06 */
	unsigned int volume:   3;  /* 00:02 000 stops ringtone*/	

} busylight_jump_step_t;
```

### Device Operation

#### Activating With a RGB Color

#### Turning the Light Off

### Observations

### Functionality Wishlist


[0]: https://busylight.com
[1]: https://github.com/JnyJny/busylight/blob/master/docs/assets/BusyLightLogo.png
[H]: https://github.com/libusb/hidapi
[Embrava]: https://github.com/JnyJny/busylight/blob/master/docs/devices/embrava.md
[CHKSUM]: https://wikipedia.com/checksum