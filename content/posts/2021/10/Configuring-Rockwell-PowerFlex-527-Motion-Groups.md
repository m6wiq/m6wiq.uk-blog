---
title: "Configuring Rockwell PowerFlex 527 Motion Groups"
date: 2021-10-23T18:47:22+01:00
draft: true
author: "M6WIQ"
---

## Hardware

1) Add the PowerFlex 527-STO CIP Safety PowerFlex 527 AC Drive module.
2) Leave the name of the module until the motion group is created.
3) Modify the IP address if required.

Note: If the Master base program has been used as the PLC template, which includes 1 drive setup in IO tree and motion group, these items need to be renamed or deleting before adding a new drive from the Master base program copy. If this is not done the parameters for the drive are lost when copying from another project.
