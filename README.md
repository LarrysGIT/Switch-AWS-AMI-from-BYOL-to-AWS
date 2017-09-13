
# This is the easiest way to switch your AMI (Windows) licensing type from BYOL to AWS

* Personal experience

* Not official

## aws ec2 import-image --license-type [BYOL|AWS]

###  There are 2 ways to determine your image is using BYOL or AWS

* When you using `aws ec2 import-image` cli, there is a parameter called `--license-type`, it's `BYOL` by default. Unfortunately, there is no further way to see the property again.

* Launch a 2012 instance, see if the instance is activated by AWS KMS `169.254.169.250:1688` or `169.254.169.251:1688`, if yes, your AMI is AWS managed license, to double confirm, you can run `slmgr /rearm` to clear activation status and reboot see if the instance is activated again. This by default is a automatic behivor controlled by `ec2config` service, you can see its logs in `C:\Program Files\Amazon\Ec2ConfigService\Logs\Ec2ConfigLog.log` to know how it works. Below is a peice of logs shows this instance is launched from a BYOL AMI, so ec2config will not activate it.

```
Ec2WindowsActivate: Ec2WindowsActivate plugin has skipped activation since instance will be configured with BYOL.
```

* As for 2016, `ec2config` has been replaced by `ec2launch`, I didn't put much attention yet, I think still can confirm by AWS KMS hosts in most cases.

## Then, how to switch from BYOL to AWS and keep all my customnized OS settings?

* There is no official way to do it as I know so far.

* You can run `aws ec2 import-image --license-type AWS` to import your image again. Or, use following trick.

### Trick

* Launch a instance from your BYOL AMI, refer as `i-BYOL`, from ec2config logs you can see Ec2WindowsActivate doesn't work because it's BYOL.

* Do nothing and shutdown `i-BYOL` directly

* Find the volume of `i-BYOL` and create a snapshot, refer as `snapshot-BYOL`

* Terminate `i-BYOL`

* Launch a instance from AWS base AMI, refer as `i-AWS`, no need to wait it, just stop it immediately.

* Find the volume of `i-AWS`, detach and delete it.

* Go to snapshot page, find `snapshot-BYOL`, create a new volume from it (note the availibility zone should be the same to `i-AWS`), refer as `vol-BYOL`.

* Delete `snapshot-BYOL`

* Go to volume page, attach `vol-BYOL` to `i-AWS` at mountpoint `/dev/sda1`

* Start `i-AWS` and you will see it is activated by AWS KMS.

* Do anything you want and use ec2config or ec2launch to sysprep the server, create a AMI from the instance, the AMI will be AWS license managed and keep all customnized OS settings.

* If ec2launch or ec2config or SSMAgent don't start, please check routing by `route print` to `169.254.169.xxx` and make sure it's correct, AWS services are heavily relay on these dynamically configured link-local addresses.

-- Larry.Song@outlook.com



