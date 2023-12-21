Dzzoffice-2.01 background SQL injection vulnerability

Dzzoffice introduction:

DzzOffice is an open source office suite suitable for enterprises and teams to build their own enterprise collaborative office platforms similar to "Google Enterprise Application Suite" and "Microsoft Office 365".

project address: https://github.com/zyx0814/dzzoffice/

Vulnerability description:

There is a SQL injection vulnerability in the backend "Network Disk" module of Dzzoffice-2.01 version, through which information in the database can be obtained.

Vulnerability analysis:

In the "filelist" method, some parameters are first judged and assigned. Although these parameters are obtained from the data packet, they have been reassigned and cannot be controlled by our input.

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/de7a5c95-1366-4fa1-b532-cb0a3c98fb3c">

Next, determine whether "doobj" and "do_obj" are empty. If not, use the "trim()" function to remove leading and trailing spaces, tabs, etc. and then store them in the array "$condition".This is where the two parameters are where we can control the input, and there are no other filtering restrictions. Regarding the obtained "uids" parameter, it seems that it can be used at present.The "startdate" and "enddate" parameters are converted into timestamps through the "strtotime()" function and then stored in the array "$condition"; they cannot be used.Then call the fetch_all_event($start, $limit, $condition, $ordersql, true) function. After the above analysis, only "doobj", "do_obj" and "uids" in the "$condition" array are controllable.

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/c8a001ae-b34b-43c1-a58d-881795a9e9f3">

Follow up the fetch_all_event() function.

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/46a94c51-252e-4974-885f-1b223fa8b2c6">

The parameters that can be controlled are placed in the "$condition" array, and the relevant places can be directly found for further analysis.
It can be seen that in the code for processing "$condition", different connection parameters are used based on the different values taken out, and then assigned to "$wheresql", and finally spliced into the SQL statement for execution. There is no correctness in this process. Take the value from "$condition" and do any filtering.

<img width="415" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/b8051b41-8a2a-4dde-9792-253b6db5e14d">

Vulnerability recurrence:

Since the vulnerability requires user login, we may need a working account and password.
Log in first.

![image](https://github.com/EternalGemini/dzz/assets/98577439/1b9fdede-32c2-4be2-a0f5-b205f2fbaf91)

Then find where the vulnerability occurs.

![image](https://github.com/EternalGemini/dzz/assets/98577439/dcac45ca-af4d-417a-a162-2f1217e5c4f4)

Capture the data packet and modify "doobj" and "do_obj" as verification codes.

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/247413a7-3687-48af-9338-ce77117c4407">
<img width="415" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/d7bb60e2-c35e-4bf8-b7cd-c4984739cd22">

At this point, the repetition is complete.

The affected version:

DzzOffice2.01
