---
template: post
title: NFT hat build guide
slug: interactive-nft-hat-build-guide
draft: true
date: 2022-04-25T15:05:32.228Z
description: A guide on how you can make your own NFT hat
category: products
tags:
  - products
---


**Things you need:**
- a baseball or snapback hat that is size-adjustable in the back
- a [Raspberry Pi 12.5W Micro USB Power Supply](https://www.raspberrypi.com/products/micro-usb-power-supply/) (for Pi Zero 2 W, 3 B+ and earlier) - EU for £6.00
- a [NOOBS microSD card](https://shop.pimoroni.com/products/noobs-32gb-microsd-card-3-1?variant=31703694245971) (3.3) - 32GB for £7.50
- a [Short Pi standoffs](https://shop.pimoroni.com/products/short-pi-standoffs-for-hyperpixel-round?variant=39384564236371) (for Hyperpixel Round) for £2.25
- a [HyperPixel 2.1 Round](https://shop.pimoroni.com/products/hyperpixel-round?variant=39381081882707) - Hi-Res Display for Raspberry Pi - Touch for £40.00
- a [Raspberry Pi Zero 2 W](https://shop.pimoroni.com/products/raspberry-pi-zero-2-w?variant=39493046075475) for £11.00
- a [TNTOR - 2500 mAh power bank](https://www.amazon.de/TNTOR-Powerbank-Externer-Sicherer-Ladechip/dp/B082WW9T6P), super ultra thin design [only 4 mm] built-in cable mini power bank charger - (matte black, micro USB + USB C adapter) - 17 euro or similar 5V power bank that has a small form function
- a [GPIO Hammer Header (Solderless) Male + Female + Installation Jig](https://shop.pimoroni.com/products/gpio-hammer-header?variant=35643318026)
- a micro sd card reader for your computer
![IMG_2587.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4b1e267-2be9-4c4b-a830-297063921fcc/IMG_2587.jpeg)
## Step 1: Attach the GPIO header to the raspberry pi
I used this video to guide me - although it’s a bit lengthy. [https://www.youtube.com/watch?v=VDJ2-feg2lk&feature=emb_title](https://www.youtube.com/watch?v=VDJ2-feg2lk&feature=emb_title)
## Step 2: Attach the screen using the standoffs
Use the guide here; [https://shop.pimoroni.com/products/hyperpixel-round?variant=39381081882707](https://shop.pimoroni.com/products/hyperpixel-round?variant=39381081882707)
![IMG_2599.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26641294-d897-428e-a279-312361c201c0/IMG_2599.jpeg)
## Step 3: Insert the microSD card/ Plug in the Pi to either the wall or to the powerbank
## Step 4: Setup the pi software
I did this headlessly, via this guide; [https://desertbot.io/blog/headless-raspberry-pi-4-ssh-wifi-setup](https://desertbot.io/blog/headless-raspberry-pi-4-ssh-wifi-setup)
## Step 5: Install hyperpixel touch libraries for touch interface
Follow the steps here; [https://github.com/pimoroni/hyperpixel2r-python](https://github.com/pimoroni/hyperpixel2r-python)
## Step 6: Running
Connect to your pi `ssh pi@raspberrypi.local`, and enter your password
Run hyperpixel touch mode in the background
```jsx
cd Pimoroni/hyperpixel2r/examples/
sudo python3 uinput-touch.py &
```
Opens a browser full screen to the specified url with touch working. In this case my repl
```jsx
DISPLAY=:0 chromium-browser --start-maximized --kiosk https://davids-hat.davidfurlong1.repl.co/
```
You can fork my [repl.it](http://repl.it) here [https://replit.com/@DavidFurlong1/Davids-hat#src/App.tsx](https://replit.com/@DavidFurlong1/Davids-hat#src/App.tsx)
![IMG_2604.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f37de1ce-c48b-4907-81e3-5b9e37371538/IMG_2604.jpeg)
[IMG_2597.mov](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/897a43f7-61a3-41d1-b410-874d31df9432/IMG_2597.mov)
## Step 7: Cut the hat
![IMG_2608.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7644bae6-9b67-411c-86b9-672ebfc0cd25/IMG_2608.jpeg)
![IMG_2611.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a97d1f6-fe95-45ab-94c6-8ff227438829/IMG_2611.jpeg)
![IMG_2612.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e40a023-1dc5-490b-be1d-c6f00a17e534/IMG_2612.jpeg)
## Step 8: Done 🥳
tap/click to unblack; then select a mode. You can also control it from your phone by forking my code below

[https://replit.com/@DavidFurlong1/Davids-hat-websocket-server](https://replit.com/@DavidFurlong1/Davids-hat-websocket-server)

See guide video and demo [here](https://davidfurlong.notion.site/NFT-hat-build-guide-a7f5c40b10b34b998c1143c9f2acd859)
