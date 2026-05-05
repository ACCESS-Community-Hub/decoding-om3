# Season 2

## Navigating MOM6 as a non-oceanographer: a MOM6 mini-app for GPU work.

Date: 7/05/2026.

Presenter: Jorge Gálvez Vallejo (@JorgeG94 ).

Link to repo: [hackathon_mom6_miniapp](https://github.com/JorgeG94/hackathon_mom6_miniapp)

### Brief Intro

Joining a new project in a field you did not study can be extremely daunting! But fear not, for 
fortune favors the brave. I'd say there are three options here:

- You are an oceanographer that has little knowledge of high performance computing 
- You are a high performance computing perso that has no knowledge of oceanography 
- You are a new student and don't know either

The third one naturally has the most to overcome, but they also have the most time. The first one
is simply "we have to learn how to code", the second one is "I have to learn oceanography". However, 
the second one might not need to learn a lot of oceanography since their contributions don't depend 
on them understanding the physics of the ocean dynamics. 

I am a stubborn person that doesn't like to write code I don't understand, so I basically like to 
create more work for myself.  

### Detailed notes (flow of conciousness) 

As an HPC person whose goal is to either optimize code or port it to different architectures the key 
aspect to understand is *what are the bottlenecks?* in the current code. In ocean simulations, to my 
annoyance, there is no _one_ routine that dominates 80% of the time. This makes my work harder, usually
one can quickly say "ah yes, the calculation of this physical thing takes most of the time" so one spends
the most time there. Then naturally you go ask the people that work on the codebase about the bottlenecks and
because everyone is using it for a different thing you will hear different things, this is where the oceanography
knowledge becomes useful! 

So the first step as a non-oceanographer was to ask what are the common problems/bottlenecks in the current workflows. Quickly
you start hearing "ah the barotropic solver", "the continuity bit is a bit slow", "the remapping of vertical coords, definetly". 
And, of course, you have no idea what this means. 

What is the best way to understand a gigantic codebase with thousands of lines of code? Simplify it! I.e. the creation of a mini-app. 
This is also the best thing to do for a hackathon, because you can capture algorithmic complexity, main pain points, and you 
end up with a smaller application that is easy to build, easy to verify, and low stakes if you completely break it! 

How does one create a mini-app? Usually you pick the most expensive part of the codebase and strip it for its parts. For MOM6,
as I pointed out earlier, there are no clear bottlenecks it is a bunch of routines/loops that are executed in different bits
of the code that take the most time. So if you want to get a clear picture of the _overall_ code you need to grab the 
multiple bits. In the case for this mini-app we chose the vertical and horizontal viscosities, the continuity equation, the
barotropic solver, and the coriolis force. 

How do you strip something to its parts? Well, this is when understanding a bit of the maths helps! At the very bottom, each
piece is following an algorithm to solve whatever equation it is solving. For example, the vertical viscosity is a tridiagonal
solver. You can search and learn _what_ a tridiagonal solver is and how to implement one (say in Python), then based on that 
newly acquired knowledge you look at the `MOM_vert_visc.F90` and see that around all the `if` statements, and `subroutine` calls, 
there is just a plain-old, vanilla tridiagonal solver at the bottom. Then you just yank it out, and work _backwards_ to obtain 
the input data.  

Looking at code where you understand a bit of the maths (especially in Fortran) helps narrow down your knowledge gap, since you
start noticing the `if` statements that add certain parameters, add drags, what do you do at the bottom?, etc. 

The rest is just plain old elbow grease of spending time correlating code to physics. Thankfully there are lots of lectures on 
the internet about physical oceanography, many from ARC centres of excellence. I like to listen to them as if they were 
podcasts, on my way to work or while I workout (yeah, I'm that kind of guy, helps keep pace if there's no beat). The more times
you hear "geostrophy", "non-hydrostatic Boussinesq equations", "piecewise parabolic method", "Thomas solver", "continuity", 
"vorticity", etc. coupled with your curiosity and lack of notetaking, leading to constant googling of "what is geostrophy" leads
to solidifying your knowledge base. 

Why should you invest into learning both? Well, speedups in code can come from new algorithms, and new algorthms can only be 
developed if you understand the physics! If you understand the approximations the equations are working under, you can determine
if a certan trick can be applied for a 10-20% speedup gain in the code. 



## Navier Stokes -> stacked shallow water (adiabatic)
## Generalised vertical coordinates
## Vertical Lagrangian remapping
## Pressure forces
## Coriolis term
## Pressure solver - barotropic / baroclinic split
## Timestepping / Advection schemes
## Vertical velocity diagnostics
## MEKE
## EPBL
## KPP
## Submesoscale parameterisation
## Ice shelf code
## Ocean fresh-water forcing / water balance
## How to create a new MOM6 diagnostic
## Horizontal viscosity
## Advection schemes
## Bottom drag (Callum? Luwei?)
## Opacity for shortwave penetration
## For low res: GM / Redi eddy params
## Explicit tides, self attraction and loading
## Regional boundary conditions
## Bulk formulae
## Fortran 101 - control structure, where parameters are defined, some keywords e.g. private, intent, submodule, use module etc
## How to work out how a diagnostic was calculated (general syntax + keywords to search for)
## Different initialisation options - thickness + tracers, topography (all these preset options)
## Coord config vs use_regridding (layer vs ale mode) - how to change your vertical coordinate or target coordinate and what it means
## Regridding of diagnostics and vanished layer output - how to change in diag_table, what it means for output
## Equations of state and different types of salinity, temperature in initialisation, model running and output
## What does the sea ice model do for the ocean ?
## MOM5/MOM6 code differences?
## Shear mixing
## Internal tide mixing
## Lee wave mixing
## Bottom boundary mixing
## Background mixing
