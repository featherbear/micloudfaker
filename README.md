# micloudfaker

Emulates just enough of the Xiaomi miIO "cloud" servers `ott.io.mi.com` and `ot.io.mi.com` to fix
[the really stupid problem of the `qmi.powerstrip.v1` Smart Power Strip not updating status info when cut off from the internet, even returning *broken JSON* when it hasn't ever connected since bootup](https://github.com/rytilahti/python-miio/issues/334).
"Just enough" being replying to client hellos and ping-pongs.

(Yeah, that's it, literally. This is probably one of the silliest firmware behaviors ever.
The device doesn't actually need anything from the servers, but just this "get status" feature is broken in a weird way when the servers are unavailable. Wat?!)

Huge thanks to the authors of [Dustcloud](https://github.com/dgiese/dustcloud) and [python-miio](https://github.com/rytilahti/python-miio).

PSA: Prefer devices with [preflashed open source firmware](https://templates.blakadder.com/preflashed.html) or [official OTA custom firmware installation](https://tasmota.github.io/docs/Sonoff-DIY/)!

## Features

- 100% Safe Rust
- no external crate dependencies, std library only
- no configuration
- no command line arguments

## Usage

- `cargo build --release`, take the result at `target/release/micloudfaker`
- run that binary on some server, why not directly on your gateway/firewall
- configure DNS overrides:
	- e.g. on OPNsense, Unbound host overrides configured via the admin UI work fine
	- at least `ott.io.mi.com` and `ot.io.mi.com` both should return some particular address
		- just do this as a wildcard for everything under `io.mi.com`, why not
		- Dustcloud suggests `203.0.113.1` from the reserved test range
		- do not forget about this if you ever want something to communicate with the real servers again :D
- configure Destination NAT (port forwarding):
	- UDP `that-reserved-address:8053` → `your-server:8053`
	- TCP `that-reserved-address:80` → `your-server:8053`
		- to make TCP actually work on OPNsense, I've had to enable reflection on that NAT rule and tick "Automatic outbound NAT for Reflection" in "Firewall: Settings: Advanced". Otherwise I was getting an RST directly in response to every SYN-ACK, and lots of retransmissions and so on.
- power cycle the power strips (just deauthing them from Wi-Fi seems to leave DNS responses still cached, unfortunately)
- watch the output logspam
- test the affected functionality (e.g. by running `miiocli -d powerstrip status`)
- make the server run permanently (you can `2>/dev/null` because the log isn't useful when not testing/debugging)
- enjoy!

Why not return the server address directly?
Well, we use DNAT to redirect port 80 to 8053 so that we don't use a privileged port (so there's no need to mess with stuff like `mac_portacl`/Linux-capabilities or start as root).
But I've encountered an extra reason: Unbound doesn't like returning local addreses in overrides, and very quickly starts returning the real ones. o_0
(Possibly related to the same addresses being used for the DHCP-hostname-to-local-domain thing.)

## License

This is free and unencumbered software released into the public domain.  
For more information, please refer to the `UNLICENSE` file or [unlicense.org](https://unlicense.org).
