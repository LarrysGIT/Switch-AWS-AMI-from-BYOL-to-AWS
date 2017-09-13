
## This is the easiest way (I think) to switch your Windows AMI licensing type from BYOL to AWS

* Personal experience

* Not official

## aws ec2 import-image --license-type [BYOL|AWS]

* Above is the cli import your image to an AWS AMI.

* If you not sure your AMI is BYOL or AWS, there still some ways to determine,

* * When you using `aws ec2 import-image` cli, there is a parameter called `--license-type`, it's `BYOL` by default. Unfortunately, there is no further way to see or change the property again.

* * Launch a 2012 instance, see if the instance is activated by AWS KMS `169.254.169.250:1688` or `169.254.169.251:1688`, if yes, your AMI is AWS managed license, to double confirm, you can run `slmgr /rearm` to clear activation status and reboot see if the instance is activated again. This by default is a automatic behivor controlled by `ec2config` service, you can see its logs in `C:\Program Files\Amazon\Ec2ConfigService\Logs\Ec2ConfigLog.log` to know how it works. Below is a peice of logs shows this instance is launched from a BYOL AMI, so ec2config didn't activate it.

```
Ec2WindowsActivate: Ec2WindowsActivate plugin has skipped activation since instance will be configured with BYOL.
```

* As for 2016, `ec2config` is replaced by `ec2launch`, I didn't put much attention yet, however it still can be confirmed by AWS KMS hosts in most cases.

## Then, how to switch from BYOL to AWS and keep all my customnized OS settings?

* There is no official way to do it as I know so far.

* You can run `aws ec2 import-image --license-type AWS` to import your image again. Or, use following trick.

### Trick

* Go to instance page, launch an instance from your BYOL AMI, refer as `i-BYOL`, from ec2config logs you can see Ec2WindowsActivate didn't work because it's BYOL

* Do nothing and shutdown `i-BYOL`

* Go to volume page, find the volume of `i-BYOL` and create a snapshot, refer as `snapshot-BYOL`

* Now you can terminate `i-BYOL` and delete it related resources at anytime

* Go to instance page, launch a new instance from AWS base AMI, refer as `i-AWS`, no need to wait it, just stop it immediately

* Go to volume page, find the volume of `i-AWS`, detach and delete

* Go to snapshot page, find `snapshot-BYOL`, create a new volume from it (note the availibility zone a,b,c,d should be the same to `i-AWS`), refer as `vol-BYOL`

* Now you can delete `snapshot-BYOL` at anytime

* Go to volume page, attach `vol-BYOL` to `i-AWS` at mountpoint `/dev/sda1`

* Go to instance page, start `i-AWS` and login, you will see it is activated by AWS KMS.

* Do anything you want and use ec2config or ec2launch to sysprep and shutdown the server, create an AMI, the AMI will be AWS license managed and  all customnized OS settings kept.

* Tip: if ec2launch or ec2config or SSMAgent doesn't start, please check routing by `route print` to `169.254.169.xxx`, make sure it's correct, AWS services are heavily relay on these dynamically configured link-local addresses.

-- Larry.Song@outlook.com







