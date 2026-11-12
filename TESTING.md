# Testing the radios on real hardware

For whoever has the boards. You need a laptop with [PlatformIO](https://platformio.org/install/cli) (`pip install platformio`) and a USB cable per board. Nothing here has been flashed or tested over the air yet, so please send back what you see, including failures.

## 1. Get the code

Clone the three repositories into one folder, side by side (the relay builds from the node firmware's libraries):

```
git clone https://github.com/Terra-Watch/node-firmware
git clone https://github.com/Terra-Watch/gateway-firmware
git clone https://github.com/Terra-Watch/relay-firmware
```

## 2. Boards and wiring

All radios are SX1278 at 433 MHz. Fit the antenna before transmitting. 3.3 V only.

| Role | Board | Folder | Build |
|---|---|---|---|
| Gateway | ESP32-S3 devkit | gateway-firmware | `pio run` |
| Relay | NodeMCU-32S | relay-firmware | `pio run -e relay_esp32` (wiring table in README.md) |
| Station | ESP32-S3 devkit | node-firmware | `pio run -e n3g-test-s3-node-a` |

Plug one board at a time, or give the port explicitly: `pio run -e <env> -t upload --upload-port <port>` (macOS `/dev/cu.usb*`, Linux `/dev/ttyUSB0` or `/dev/ttyACM0`, Windows `COM3`). The ports written in the `platformio.ini` files are the author's and are commented out.

## 3. Flash and watch

In each folder: `pio run -e <env> -t upload`, then `pio device monitor -b <baud>` (921600 for the relay, 115200 for the others; a wrong baud shows garbage).

## 4. What to check, in order

1. **Gateway alone.** The monitor shows it started and is listening.
2. **Station to gateway, no relay.** Power the gateway and the station a few metres apart. The gateway should print `[HEARTBEAT RX] ...` and the station should log an acknowledgement.
3. **Relay in the middle.** Put the station out of the gateway's reach (or build the station with `-D TW_TX_POWER_DBM=2` in `platformio.ini`), put the relay between them. The gateway should print `[DATA RX] source=0x01 ...` and the relay should log that it forwarded.
4. **Send a message by hand.** Build the station with `-D TW_TEST_MODE=1` (the `n3g-test-s3-node-a` environment has it) and type `SEND_DATA 254 hello` in its monitor; the gateway should show the payload.
5. **Relay fails.** Unplug the relay mid-run and note what the gateway shows.

## 5. Send back

For each step: worked or not, the monitor output (copy the text), distances, and board and antenna types. If the relay or gateway shows nothing at all, run the gateway with `pio run -e diagnostic-gateway -t upload` and the relay with `-e relay_esp32_diag` and send that output.
