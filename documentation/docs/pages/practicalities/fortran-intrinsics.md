# Fortran Intrinsics

## Fortran 101 re-visited: MOM6 intrinsics

Date: 21/05/2026.

Presenter: Jorge Gálvez Vallejo (@JorgeG94).

After some careful deliberation I have decided to show a very fun bit of MOM6 that is crucial to all of our simulations! We 
know that MOM6 enforces bitwise reproducbility across runs. To achieve this there are careful safeguards while writing and reviewing
code. Bitwise reproducibility means that two answers at identical to the bits that compose them, i.e. a good example is that: 
`a + (b+c) != (a+b) + c` in terms of bit, they both resolve to `(a+b+c)` but the order of operations is different! 

So MOM6 enforces very tight checks on how the code is written, this is why if you want to refactor a routine in MOM6 
that does intermediate computations you can end up changing answers! For example one that I looked at (this routine is 
in the vertical viscosity):

```fortran
botfn = 1. / (1. + 0.09 * z2 * z2 * z2 * z2 * z2 * z2)
```

I thought, "oh my god, this is ugly", let's make this more readable by doing:

```fortran
botfn = 1. / (1. + 0.09*z2**6) 
```

There are two potential things here. One, this changes the BITS of the answer although they are 
the same number. This is because `z2**6` can be evaluated differently by the compiler! So I thought, well, 
how about:

```fortran
z2_six = z2 * z2 * z2 * z2 * z2 * z2
botfn = 1. / (1. + 0.09*z2_six)
```
You might think, well surely this is correct. Well no, because again the order of the operations has changed, the
compiler can rearrange how things are executed!

You can see now how subtle a small change can break bitwise reproducibility! So, today we are going into a super 
fun bit of the code: `MOM6/src/framework/MOM_intrinsic_functions.F90`! This file contains some routines that 
are not quite trustworthy across compilers. We will focus on the cuberoot! 

The module signature: 

```fortran
module MOM_intrinsic_functions

use iso_fortran_env, only : stdout => output_unit, stderr => error_unit
use iso_fortran_env, only : int64, real64

implicit none ; private

public :: invcosh, cuberoot
public :: intrinsic_functions_unit_tests

!...

end module MOM_intrinsic_functions
```

```fortran
!> Returns the cube root of a real argument at roundoff accuracy, in a form that works properly with
!! rescaling of the argument by integer powers of 8.  If the argument is a NaN, a NaN is returned.
elemental function cuberoot(x) result(root)
  real, intent(in) :: x !< The argument of cuberoot in arbitrary units cubed [A3]
  real :: root !< The real cube root of x in arbitrary units [A]

  real :: asx ! The absolute value of x rescaled by an integer power of 8 to put it into
              ! the range from 0.125 < asx <= 1.0, in ambiguous units cubed [B3]
  real :: root_asx ! The cube root of asx [B]
  real :: ra_3 ! root_asx cubed [B3]
  real :: num ! The numerator of an expression for the evolving estimate of the cube root of asx
              ! in arbitrary units that can grow or shrink with each iteration [B C]
  real :: den ! The denominator of an expression for the evolving estimate of the cube root of asx
              ! in arbitrary units that can grow or shrink with each iteration [C]
  real :: num_prev ! The numerator of an expression for the previous iteration of the evolving estimate
              ! of the cube root of asx in arbitrary units that can grow or shrink with each iteration [B D]
  real :: np_3 ! num_prev cubed  [B3 D3]
  real :: den_prev ! The denominator of an expression for the previous iteration of the evolving estimate of
              ! the cube root of asx in arbitrary units that can grow or shrink with each iteration [D]
  real :: dp_3 ! den_prev cubed  [C3]
  real :: r0  ! Initial value of the iterative solver. [B C]
  real :: r0_3 ! r0 cubed [B3 C3]
  integer :: itt

  integer(kind=int64) :: e_x, s_x

  if ((x >= 0.0) .eqv. (x <= 0.0)) then
    ! Return 0 for an input of 0, or NaN for a NaN input.
    root = x
  else
    call rescale_cbrt(x, asx, e_x, s_x)

    !   Iteratively determine root_asx = asx**1/3 using Halley's method and then Newton's method,
    ! noting that Halley's method onverges monotonically and needs no bounding.  Halley's method is
    ! slightly more complicated that Newton's method, but converges in a third fewer iterations.
    !   Keeping the estimates in a fractional form Root = num / den allows this calculation with
    ! no real divisions during the iterations before doing a single real division at the end,
    ! and it is therefore more computationally efficient.

    ! This first estimate gives the same magnitude of errors for 0.125 and 1.0 after two iterations.
    ! The first iteration is applied explicitly.
    r0 = 0.707106
    r0_3 = r0 * r0 * r0
    num = r0 * (r0_3 + 2.0 * asx)
    den = 2.0 * r0_3 + asx

    do itt=1,2
      ! Halley's method iterates estimates as Root = Root * (Root**3 + 2.*asx) / (2.*Root**3 + asx).
      num_prev = num ; den_prev = den

      ! Pre-compute these as integer powers, to avoid `pow()`-like intrinsics.
      np_3 = num_prev * num_prev * num_prev
      dp_3 = den_prev * den_prev * den_prev

      num = num_prev * (np_3 + 2.0 * asx * dp_3)
      den = den_prev * (2.0 * np_3 + asx * dp_3)
      ! Equivalent to:  root_asx = root_asx * (root_asx**3 + 2.*asx) / (2.*root_asx**3 + asx)
    enddo
    ! At this point the error in root_asx is better than 1 part in 3e14.
    root_asx = num / den

    ! One final iteration with Newton's method polishes up the root and gives a solution
    ! that is within the last bit of the true solution.
    ra_3 = root_asx * root_asx * root_asx
    root_asx = root_asx - (ra_3 - asx) / (3.0 * (root_asx * root_asx))

    root = descale(root_asx, e_x, s_x)
  endif
end function cuberoot
```

From Ed's last session, here we can see a couple of the things he mentioned. We have a `function` 
that is called. Functions return a single value, similar to how they work in Python. Here however, 
we have an extra `attribute` in Fortran, the `elemental` attribute. Elemental makes a routine 
callable/applicable inside an array. So the `cuberoot` routine can be used to obtain the cuberoot
of a scalar, i.e. `real :: my_scalar` or an allocatable array of n dimensions! i.e. we can call
`cuberoot` to operate on `real, allocatable :: my_array(:,:,:,:,:)` with no problems whatsoever! 

Next, we have a `call rescale_cbrt`:

```fortran
!> Rescale `a` to the range [0.125, 1) and compute its cube-root exponent.
pure subroutine rescale_cbrt(a, x, e_r, s_a)
  real, intent(in) :: a
    !< The number to be rescaled for cube-root computation [A3]
   real, intent(out) :: x
    !< The rescaled value of a in the range from 0.125 < asx <= 1.0, in ambiguous units cubed [B3]
  integer(kind=int64), intent(out) :: e_r
    !< Cube root of the exponent of the rescaling of `a`
  integer(kind=int64), intent(out) :: s_a
    !< The sign bit of a

  integer(kind=int64) :: xb
    ! Floating point value of a, bit-packed as an integer
  integer(kind=int64) :: e_a
    ! Unscaled exponent of a
  integer(kind=int64) :: e_x
    ! Exponent of x
  integer(kind=int64) :: e_div, e_mod
    ! Quotient and remainder of e in e = 3*(e/3) + modulo(e,3).

  ! Pack bits of a into xb and extract its exponent and sign.
  xb = transfer(a, 1_int64)
  s_a = ibits(xb, signbit, 1)
  e_a = ibits(xb, expbit, explen) - bias

  ! Compute terms of exponent decomposition e = 3*(e/3) + modulo(e,3).
  ! (Fortran division is round-to-zero, so we must emulate floor division.)
  e_mod = modulo(e_a, 3_int64)
  e_div = (e_a - e_mod)/3

  ! Our scaling decomposes e_a into e = {3*(e/3) + 3} + {modulo(e,3) - 3}.

  ! The first term is a perfect cube, whose cube root is computed below.
  e_r = e_div + 1

  ! The second term ensures that x is shifted to [0.125, 1).
  e_x = e_mod - 3

  ! Insert the new 11-bit exponent into xb and write to x and extend the
  ! bitcount to 12, so that the sign bit is zero and x is always positive.
  call mvbits(e_x + bias, 0, explen + 1, xb, fraclen)
  x = transfer(xb, 1.)
end subroutine rescale_cbrt
```

As we learned last week, a `subroutine` can be invoked via a `call` procedure. 
Subroutines take multiple arguments and can return multiple values at once. You can 
see this in:
```fortran
  integer(kind=int64), intent(out) :: e_r
    !< Cube root of the exponent of the rescaling of `a`
  integer(kind=int64), intent(out) :: s_a
```

`e_r` and `s_a` are both being written to inside the subroutine and being sent back. Also, 
you might notice the `pure` attribtue in the subroutine. A pure function in Fortran is a 
function with no side effects, meaning: it does not write to file/console, it does not alter
any variable outside its function body, and it does not read from or modify any external state
(such as global variables, SAVEd local variables, or I/O streams). Given the same inputs, it
must always produce the same outputs, which allows the compiler to safely call it from
within FORALL constructs, DO CONCURRENT loops, and other pure procedures. This makes it
*threadsafe*. 

Elemental functions are `pure` by design, so any `elemental` function you write is also 
`pure`, you can be verbose and say `pure elemental function cuberoot` but that's a mouthful! 

Why did I show you this? Well, you cannot go and start editing MOM6 code without understanding
the core tenet of MOM6: keep bitwise reproducbility! I hope this small entry helps you and encourages
you to go deep into the codebase! 
