---
layout: post
title: Linux Kernel Internship
date: 2018-05-14
---

I can't remember exactly how I got introduced to Free &Open Source (FOSS),
but I do remember why I became obsessed with it that I even convinced a friend during my senior year in undergrads to work with me on building a 
tech community[[1]](https://github.com/EUTS)
inspired from the FOSS community spirits.

Coming from a female only university in Saudi Arabia,
where the tech scene has been growing only recently,
I have noticed/experienced how the isolation from a tech community can result in lack of technical skills for CS/CE students.
The main reason I think is that we were missing a social community that encourages the discussion of technical topics outside of the classroom.
As such, after I've learned about FOSS, 
I believed that it had the right recipe for a solution to motivate students to share their programming interests and engage in projects that would help improve their technical skills.

I've stumbled upon Outreachy [[2]](https://www.outreachy.org/),
or as it was called before 'Open Programs for Women', during that period, 
and I thought it was my chance to actually get involved in a real open source project.

--- 

## Outreachy
 
Outreachy is a paid remote internship for three months
with the goal to recruit and encourage underrepresented groups to get involved in the development of FOSS.

One of the unique things about Outreachy is its application period. 
Unlike other programs where they require a resume and a previous experience,
Outreachy application period is an opportunity for learning in itself; where you are expected to submit at least one patch to the project you want to work on during the internship.
That might sound difficult, but what makes it feasible is 
the availability of mentors from each community who provide help and guidance for applicants, and who make the experience less terrifying.

At the beginning of each application round, Outreachy announces a list of participating organizations with their projects
(ex. Ceph, Debian, GNOME).
The applicant has to choose specific projects to work on during the application period.

--- 

## Linux Kernel

Since I've always been curious about how the hardware and software interact with each other,
I choose to work with the Linux Kernel community as I found it an opportunity to poke around the kernel source code.


Contrary to the common belief, 
it is actually easy to start contributing to the Linux Kernel as they have a detailed 
tutorial [[3]](https://kernelnewbies.org/Outreachyfirstpatch)
on how to submit a patch starting from 
how to install and compile the kernel, how to look for issues in the code, and even how to write a proper commit message.

Just from the application period three years ago, 
I've learned many things and got presented with many opportunities later 
which I believe wouldn't have been possible without the initial push of that experience.

Although I wasn't selected back in 2015 summer round,
after observing how much I've learned within only two months,
I made it a point to check out the available projects and tasks for each round
and to at least contribute a few patches during each application period,
even though I knew that I would be pre-occupied with school or work during the internship.

My patches submitted until now only from the applications rounds can be found [here](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/log/?showmsg=1&qt=author&q=haneen).
There are maybe more than 60 patches, but I'd say I am only proud of around 10 of them. 
The others although are trivial, like fixing a coding style issue,
were indeed essential in helping me to become familiar with the various parts of the Linux Kernel as well as the contribution process.
After each patch submitted, I've gained more confidence and learned new thing about C, or the kernel, or how to use a new tool in the process.

I owe to the Linux kernel mentors for their patience and constructive feedback,
with special thanks to Julia Lawall who was supportive during and after the application period since I first started.

I am happy to say that, this summer, I will be working with the Linux Kernel as an Outreachy intern!

---

## Internship Project

I'll be working, with another intern from
GSoC [[4]](https://summerofcode.withgoogle.com/),
on the GPU subsystem [[5]](https://01.org/linuxgraphics/gfx-docs/drm/gpu/introduction.html) of the Linux Kernel,
under Daniel Vetter, Sean Paul, and Gustavo Padovan supervision.
Specifically, we would be working on building a Virtual Kernel Mode-Setting driver.
The main benefits of having a virtual KMS is that 
it could be used for testing
as well as
running X or Wayland (or similar) on a headless machine and still be able to use the GPU.
Thus, enabling a virtual display without the need for hardware display capability.

---

During this summer, I'll document my journey here. 
After all, I started this blog three years ago after getting inspired by previous Outreachy interns.

---

### Refrences:
[1] [Effat University Tech Society](https://github.com/EUTS)

[2] [Outreachy Official Website](https://www.outreachy.org/)

[3] [Linux Kernel Newbies Website](https://kernelnewbies.org/Outreachyfirstpatch)

[4] [Google Summer of Code, GSoC](https://summerofcode.withgoogle.com/)

[5] [GPU/DRM Subsystem](https://01.org/linuxgraphics/gfx-docs/drm/gpu/introduction.html)
