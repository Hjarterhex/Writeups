# Wipeout!

Solves: 0 <br/> Points: 300

## Challenge description

We have received indications that the suspects phone has been previously used and was completely wiped before being set up again in its current state. What was the last time the phone was wiped? Flag format is date and time (local Swedish time). Example HHCTF{2019-07-16 15:55}

## Solution

The file .obliterated found in /private/var/root is a 0 byte file but its creation time is the time of the first reboot after complete wipe and thus is the best indication of wipetime (even though the actual wiping happens over the course of the few minutes previous to this time). The time for creation in our case is 13:41 24 March 2022. The time can be confirmed by looking at the log-files in First boot according to /private/var/root/Library/Logs/MobileContainerManager which states first boot initialization was at 05:41 on March 24. The very first time stated is defaulted to display in UTC -8 and not the local time. As Sweden is UTC +1 or +2 depending on DST, and California is also UTC -8 or -7 depending on DST the date must be considered to determine if the times match. On 24 March 2022 California had exited daylight savings time (so summer) meaning UTC -7, but Sweden was still in Daylight savings time(so winter) meaning UTC +1, the differing hours are 8. By adding 8 hours to the log file time 05:41 we get 13:41 which is confirming the local time of the obliterated file.

**Flag:** `HHCTF{2022-03-24 13:41}`
