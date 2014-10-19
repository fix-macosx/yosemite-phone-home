# E.T. Phone Home?

This repository provides a corpus of network communications automatically sent
to Apple by OS X Yosemite; we're using this dataset
to [explore how Yosemite shares user data with Apple](https://fix-macosx.com).

The provided data was collected using our
[Net Monitor](https://github.com/fix-macosx/net-monitor) toolkit; more information regarding usage and methodology is provided below.

## Examples

The following occur with all privacy options enabled -- including disabling
analytics (i.e., *Diagnostics and Usage Data*).

**About this Mac**

When the user selects 'About this Mac' from the Apple menu, Yosemite phones home and `s_vi`, a unique analytics identifier, is [included in the request](eff-user-r0/Applications/Utilities/System Information.app/Contents/MacOS/System Information/20141019T192957Z-effuser-[172.16.174.146]:49495-[23.3.12.195]:80.log). (`si_vi` is used by [Adobe/Omniture's analytics software](http://microsite.omniture.com/t2/help/en_US/whitepapers/cookies/cookies_analytics.html#concept_98805569FE284595B34A7684647D7C71__section_5D50A078DE444D12B7D927D68FF3B679)).

If we search the logs for the cookie value, we can find:

* Where the identifying cookie [was first set](eff-user-r0/System/Library/Frameworks/WebKit.framework/Versions/A/XPCServices/com.apple.WebKit.Networking.xpc/Contents/MacOS/com.apple.WebKit.Networking/20141019T192908Z-effuser-[172.16.174.146]:49491-[66.235.139.206]:80.log) -- when the user visited <http://www.apple.com> in Safari, with an expiration of two years.
* Where else the cookie is sent to Apple -- for example, when both [Spotlight](eff-user-r0/System/Library/CoreServices/Spotlight.app/Contents/XPCServices/com.apple.metadata.SpotlightNetHelper.xpc/Contents/MacOS/com.apple.metadata.SpotlightNetHelper/20141019T200316Z-effuser-[172.16.174.146]:49166-[17.254.32.16]:80.log) and [Help](eff-user-r0/System/Library/CoreServices/HelpViewer.app/Contents/MacOS/HelpViewer/20141019T193022Z-effuser-[172.16.174.146]:49539-[96.17.236.244]:443.log) phone home.

**DuckDuckGo for Privacy**

Having read DuckDuckGo's privacy statements, you might decide to switch Safari's default search to DuckDuckGo. If we enter a new search in Safari, we can then search the logged data to see who the search terms are actually sent to.

The logs show that Safari searches are [still sent to Apple](eff-user-r0/Applications/Safari.app/Contents/MacOS/Safari/20141019T204534Z-effuser-[172.16.174.146]:49700-[17.249.89.247]:443.log), *even when selecting DuckDuckGo as your search provider, and 'Spotlight Suggestions' are disabled*.

**Non-Cloud Mail Account**

When setting up a new Mail.app account for the address `admin@fix-macosx.com`, which is hosted locally, searching the
logs for "fix-macosx.com" shows that Mail quietly [sends the domain entered by the user to Apple](eff-user-r0/Applications/Mail.app/Contents/MacOS/Mail/20141019T193338Z-effuser-[172.16.174.146]:49713-[17.134.62.228]:443.log), too.

# Methodology, Usage, and Caveats

Two different datasets are provided; these were generated in independent VMs
with fresh installs of Mac OS X Yosemite:

* `eff-user-r0`
	* All data sharing options disabled.
	* Location services disabled.
	* iCloud not used.
	* No Apple ID used.
	* DuckDuckGo selected as Safari search engine

* `icloud-user-r0`
	* Installed with all default options, including sending of "Diagnostics and Usage Data".
	* iCloud and most iCloud features enabled, including iCloud drive.

All TCP/SSL connections are logged with one file per connection: `<application path>/<iso 8601 time>-<username>-<src addr>-<dest-addr>.log`
Non-TCP traffic (such as UDP, ICMP) is logged in pcap format in `udp-monitor/*.pcap`.

## Caveats

* This data was collected over the course of a few hours, and with only minimal interaction with the system and applications. It is
not a complete representative set of all data potentially collected by Yosemite; for example:
** `icloud-user-r0` dataset does not contain the diagnostics data periodically sent to Apple.
** Cursory usage means that application-specific logs are not representative -- e.g., when setting up a Mail account, we only entered information on the first screen.
* Correlation of sockets with file system executable paths is reasonably accurate; actual correspondance should be sanity checked (we've
seen cases where `proc_pidpath()` returned paths for processes that could not be running).
* TLS traffic using client certificates cannot be captured in plaintext by default. For
example, NM captures the key exchange performed by apsd (Apple Push Services Daemon),
that establishes a client certificate, but NM can't transparently sniff future communications
protected by that certificate without the addition of apsd-specific protocol handling.
* Not all traffic is logged in plaintext, so the lack of a match on a search should not be treated as conclusive; it may be necessary to
decode data that was encoded for transmission via URL encoding, base64, protobuf, etc.

## Contributing

Help is requested in all of the following areas:

* Finding and [documenting](https://github.com/fix-macosx/fix-macosx/issues) privacy issues.
* Enhanced automated dataset visualization/decoding.
* Adding application-specific support for processes using client-certificates to [SSLsplit](https://github.com/fix-macosx/sslsplit).
* Automated (re-)generation of the datasets (e.g, scripting installation and application use).
* Using [net-monitor](https://github.com/fix-macosx/net-monitor) to gather data from AirDrop, Handoff, and other technologies that are difficult to run in a VM environment.
* Exploring work-arounds (e.g., sandboxing, firewalling).
