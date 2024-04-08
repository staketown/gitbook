# IBC Monitoring

## Introduction
Monitoring tools that tracks stuck packets, acknowledgements and clients expiration and notify 
about any of above incidents for each path that is being tracked to discord channel.

## Chains integrated
* [Celestia](/docs/mainnet/celestia)
* [Umee](/docs/mainnet/umee)

## How it works

We provide ibc monitoring tool based on custom ibc exporter, prometheus and alertmanager that post
about any incidents trigger with specified rules over provided discord webhook to specified channel.
Tool tracks the following:
 - client is about to be expired (less than 20h) on any chain of ibc path;
 - client has been expired;
 - stuck acknowledgements on any channel of ibc path;
 - stuck packets on any channel of ibc path;

<div>
<img src="../.gitbook/assets/ibc-1.png" alt="alert stuck packets" width="90%"/> 
<img src="../.gitbook/assets/ibc-2.png" alt="resolved stuck packets" width="90%"/>
</div>
<div>
<img src="../.gitbook/assets/ibc-3.png" alt="alert stuck acknowledgments" width="90%"/>
<img src="../.gitbook/assets/ibc-4.png" alt="resolved stuck acknowledgments" width="90%"/>
</div>
<div>
<img src="../.gitbook/assets/ibc-5.png" alt="alert client above to expire" width="90%"/>
<img src="../.gitbook/assets/ibc-6.png" alt="resolved client above to expire" width="90%"/>
</div>
<div>
<img src="../.gitbook/assets/ibc-7.png" alt="alert client is expired" width="90%"/>
</div>

## How to integrate?

Interested in integration such tool? Don't hesitate to contact us:

Email: **hello@stake-town.com**<br/>
Discord: **hottochelli**<br/>