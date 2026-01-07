---
title: 'Raspberry Pi notes'
date: '10:13 15-10-2022'
taxonomy:
    category:
        - docs
---

## Notes

### Raspberry Pi 3B LED
```
# Turn off Power LED
dtparam=pwr_led_trigger=none
dtparam=pwr_led_activelow=off

# Turn off Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off

# Turn off Ethernet ACT LED
dtparam=eth_led0=14

# Turn off Ethernet LNK LED
dtparam=eth_led1=14
```

### Raspberry Pi 4B LED
```
# Turn off Power LED
dtparam=pwr_led_trigger=none
dtparam=pwr_led_activelow=off

# Turn off Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off

# Turn off Ethernet ACT LED
dtparam=eth_led0=4

# Turn off Ethernet LNK LED
dtparam=eth_led1=4
```