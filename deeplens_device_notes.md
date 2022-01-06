# Boot Timings

0. power off
1. push button
    * yellow light w/ power indicator
    * yellow->blue wifi indicator
    * screen display on
2. two blue lights : power and wifi
    * all three lights are blue power, wifi, dot indicator
    * ssh login top shows python 2.7 process 
       * blink twice: video process not running
3. aws console will display as online
4. top output 
    * python 2.7 process as number one process
    * load avg 2.95


top shows these as top two processes
python interpeter
video-server.py
load avg 2.96


Device logs are in:

/opt/awscam/greengrass/ggc/var/log/user/us-east-1/265627204426


Timings on deply new function

0. edit project, 
    * choose new function version, 
    * click deploy
1. click deploy to device from aws deeplens console,
    * select device,
    * click to deploy
2. some time into this minute, the python process disappears
3. the exe process shows up during the status message in aws console mentioning greengrass


Adjusted cloud code to have a higher threshold.

Two images: One without safety hat and one with safety hat.

Without safety hat info

```
bucker// path davisjf-worker-safety // persons/1_24/13_55/1611514520_0.jpg

2021-01-24T13:58:21.393-05:00	person {'Width': 0.7009846568107605, 'Height': 0.8564843535423279, 'Left': 0.13481014966964722, 'Top': 0.14088411629199982}

2021-01-24T13:58:21.907-05:00	Person(s): 1

2021-01-24T13:58:21.907-05:00	Person(s) With Safety Hat: 0

2021-01-24T13:58:21.907-05:00	Person(s) Without Safety Hat: 1
```

This image does not have a hard hard hat in frame.

With safety hat info

```
bucker// path davisjf-worker-safety // persons/1_24/13_58/1611514702_0.jpg

2021-01-24T13:58:25.421-05:00	person {'Width': 0.8000633716583252, 'Height': 0.8899438977241516, 'Left': 0.13117554783821106, 'Top': 0.09842915087938309}

2021-01-24T13:58:25.421-05:00	hardhat {'Width': 0.2521851062774658, 'Height': 0.39033666253089905, 'Left': 0.45245492458343506, 'Top': 0.08758512884378433}

2021-01-24T13:58:25.893-05:00	Person(s): 1

2021-01-24T13:58:25.893-05:00	Person(s) With Safety Hat: 1

2021-01-24T13:58:25.893-05:00	Person(s) Without Safety Hat: 0

```

Notes on axis and dimensions

Width             Height     Left          Top            Label
0.8                0.9        0.13         0.1            person
0.252              0.4        0.45         0.09           hardhat

   |
   | 
---+----
   | xxxx
   |    
