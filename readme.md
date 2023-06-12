# welcome octave solver


![math](/matrix/bin/russian/client/images/math.jpeg)

20.1 Solvers

Octave can solve sets of nonlinear equations of the form

F (x) = 0

using the function fsolve, which is based on the MINPACK subroutine hybrd. This is an iterative technique so a starting point must be provided. This also has the consequence that convergence is not guaranteed even if a solution exists.

: fsolve (fcn, x0, options)
: [x, fvec, info, output, fjac] = fsolve (fcn, …)

    Solve a system of nonlinear equations defined by the function fcn.

    fcn should accept a vector (array) defining the unknown variables, and return a vector of left-hand sides of the equations. Right-hand sides are defined to be zeros. In other words, this function attempts to determine a vector x such that fcn (x) gives (approximately) all zeros.

    x0 determines a starting guess. The shape of x0 is preserved in all calls to fcn, but otherwise it is treated as a column vector.

    options is a structure specifying additional options. Currently, fsolve recognizes these options: "FunValCheck", "OutputFcn", "TolX", "TolFun", "MaxIter", "MaxFunEvals", "Jacobian", "Updating", "ComplexEqn" "TypicalX", "AutoScaling" and "FinDiffType".

    If "Jacobian" is "on", it specifies that fcn, called with 2 output arguments also returns the Jacobian matrix of right-hand sides at the requested point. "TolX" specifies the termination tolerance in the unknown variables, while "TolFun" is a tolerance for equations. Default is 1e-7 for both "TolX" and "TolFun".

    If "AutoScaling" is on, the variables will be automatically scaled according to the column norms of the (estimated) Jacobian. As a result, TolF becomes scaling-independent. By default, this option is off because it may sometimes deliver unexpected (though mathematically correct) results.

    If "Updating" is "on", the function will attempt to use Broyden updates to update the Jacobian, in order to reduce the amount of Jacobian calculations. If your user function always calculates the Jacobian (regardless of number of output arguments) then this option provides no advantage and should be set to false.

    "ComplexEqn" is "on", fsolve will attempt to solve complex equations in complex variables, assuming that the equations possess a complex derivative (i.e., are holomorphic). If this is not what you want, you should unpack the real and imaginary parts of the system to get a real system.

    For description of the other options, see optimset.

    On return, fval contains the value of the function fcn evaluated at x.

    info may be one of the following values:

    1

        Converged to a solution point. Relative residual error is less than specified by TolFun.
    2

        Last relative step size was less that TolX.
    3

        Last relative decrease in residual was less than TolF.
    0

        Iteration limit exceeded.
    -3

        The trust region radius became excessively small. 

    Note: If you only have a single nonlinear equation of one variable, using fzero is usually a much better idea.

    Note about user-supplied Jacobians: As an inherent property of the algorithm, a Jacobian is always requested for a solution vector whose residual vector is already known, and it is the last accepted successful step. Often this will be one of the last two calls, but not always. If the savings by reusing intermediate results from residual calculation in Jacobian calculation are significant, the best strategy is to employ OutputFcn: After a vector is evaluated for residuals, if OutputFcn is called with that vector, then the intermediate results should be saved for future Jacobian evaluation, and should be kept until a Jacobian evaluation is requested or until OutputFcn is called with a different vector, in which case they should be dropped in favor of this most recent vector. A short example how this can be achieved follows:

    function [fvec, fjac] = user_func (x, optimvalues, state)
    persistent sav = [], sav0 = [];
    if (nargin == 1)
      ## evaluation call
      if (nargout == 1)
        sav0.x = x; # mark saved vector
        ## calculate fvec, save results to sav0.
      elseif (nargout == 2)
        ## calculate fjac using sav.
      endif
    else
      ## outputfcn call.
      if (all (x == sav0.x))
        sav = sav0;
      endif
      ## maybe output iteration status, etc.
    endif
    endfunction

    ## …

    fsolve (@user_func, x0, optimset ("OutputFcn", @user_func, …))

    See also: fzero, optimset. 

The following is a complete example. To solve the set of equations

-2x^2 + 3xy   + 4 sin(y) = 6
 3x^2 - 2xy^2 + 3 cos(x) = -4

you first need to write a function to compute the value of the given function. For example:

function y = f (x)
  y = zeros (2, 1);
  y(1) = -2*x(1)^2 + 3*x(1)*x(2)   + 4*sin(x(2)) - 6;
  y(2) =  3*x(1)^2 - 2*x(1)*x(2)^2 + 3*cos(x(1)) + 4;
endfunction

Then, call fsolve with a specified initial condition to find the roots of the system of equations. For example, given the function f defined above,

[x, fval, info] = fsolve (@f, [1; 2])

results in the solution

x =

  0.57983
  2.54621

fval =

  -5.7184e-10
   5.5460e-10

info = 1

A value of info = 1 indicates that the solution has converged.

When no Jacobian is supplied (as in the example above) it is approximated numerically. This requires more function evaluations, and hence is less efficient. In the example above we could compute the Jacobian analytically as

function [y, jac] = f (x)
  y = zeros (2, 1);
  y(1) = -2*x(1)^2 + 3*x(1)*x(2)   + 4*sin(x(2)) - 6;
  y(2) =  3*x(1)^2 - 2*x(1)*x(2)^2 + 3*cos(x(1)) + 4;
  if (nargout == 2)
    jac = zeros (2, 2);
    jac(1,1) =  3*x(2) - 4*x(1);
    jac(1,2) =  4*cos(x(2)) + 3*x(1);
    jac(2,1) = -2*x(2)^2 - 3*sin(x(1)) + 6*x(1);
    jac(2,2) = -4*x(1)*x(2);
  endif
endfunction

The Jacobian can then be used with the following call to fsolve:

[x, fval, info] = fsolve (@f, [1; 2], optimset ("jacobian", "on"));

which gives the same solution as before.

: fzero (fun, x0)
: fzero (fun, x0, options)
: [x, fval, info, output] = fzero (…)

    Find a zero of a univariate function.

    fun is a function handle, inline function, or string containing the name of the function to evaluate.

    x0 should be a two-element vector specifying two points which bracket a zero. In other words, there must be a change in sign of the function between x0(1) and x0(2). More mathematically, the following must hold

    sign (fun(x0(1))) * sign (fun(x0(2))) <= 0

    If x0 is a single scalar then several nearby and distant values are probed in an attempt to obtain a valid bracketing. If this is not successful, the function fails.

    options is a structure specifying additional options. Currently, fzero recognizes these options: "FunValCheck", "OutputFcn", "TolX", "MaxIter", "MaxFunEvals". For a description of these options, see optimset.

    On exit, the function returns x, the approximate zero point and fval, the function value thereof.

    info is an exit flag that can have these values:

        1 The algorithm converged to a solution.
        0 Maximum number of iterations or function evaluations has been reached.
        -1 The algorithm has been terminated from user output function.
        -5 The algorithm may have converged to a singular point. 

    output is a structure containing runtime information about the fzero algorithm. Fields in the structure are:

        iterations Number of iterations through loop.
        nfev Number of function evaluations.
        bracketx A two-element vector with the final bracketing of the zero along the x-axis.
        brackety A two-element vector with the final bracketing of the zero along the y-axis. 

    See also: optimset, fsolve. 