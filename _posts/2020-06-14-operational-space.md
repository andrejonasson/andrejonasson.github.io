---
layout: post
title: Operational Space Formulation
---
### Preliminaries
Let $q$ be a set of $n$ generalized coordinates

<div>
\begin{equation}
A(q)\ddot q + k(q, \dot q) = \tau \label{eq:jdyn}
\end{equation}
</div>

where $A(q)$ is the inertia matrix, $k(q, \dot q)$ is a collected term for the gravity, centrifugal and Coriolis torques and $\tau$ is the external torques (e.g. motor torques). The inertia matrix $A$ is positive definite since the kinetic energy $\dot q^T A \dot q$ is greater than zero for any $\dot q \ne 0$, this fact will be used later.

### Operational Space

Say there is an end effector (operational space) that we wish to control with $m$ degrees of freedom and this end effector is part of a manipulator with $n$ joints where $m \lt n$. Then if a joint position exists that expresses an end effector position, then there are infinite such joint positions with the same end effector position. Redundancy..

The same holds true for force and torque. There are infinite torque solutions in joint space corresponding to a force in operational space. A particular torque solution is the force torque relationship $\tau = J^TF$ where $J$ is the Jacobian of the forward kinematics from the generalized coordinates to the operational space 
<div>
\begin{equation}
\dot x = J(q) \dot q \label{eq:xdotjqdot}
\end{equation}
</div>
where $x$ is the operational space coordinates. 

We want to derive the equations governing the dynamics of the operational space. Similarly to how $\eqref{eq:jdyn}$ relates torques to changes in the configuration of the joints we want a relationship from operational force to the configuration of the operational coordinates (e.g. end effector position and orientation). 

By the derivative product rule applied to $\eqref{eq:xdotjqdot}$ we can derive a relationship between operational acceleration and joint space dynamics. This equation is key to deriving the operational space dynamics
<div>
\begin{equation*}
\ddot x = \dot J \dot q + J \ddot q
\end{equation*}
</div>
Now use $\eqref{eq:jdyn}$ to replace $\ddot q$
<div>
\begin{equation*}
\ddot x = \dot J \dot q + J A^{-1}(\tau-k)
\end{equation*}
</div>

<div>
\begin{equation}
= \dot J \dot q + J A^{-1}(J^TF-k) \label{eq:opdynintermediate}
\end{equation}
</div>

We can now form a dynamics equation in operational space, similar in form to $\eqref{eq:jdyn}$. We rearrange $\eqref{eq:opdynintermediate}$ such that operational force is by itself on the right hand side, giving

<div>
\begin{equation}
\Lambda(q) \ddot x + h(q,\dot q) = F \label{eq:opdyn}
\end{equation}
</div>
<div>
\begin{equation*}
h(q,\dot q) = \Lambda(q) J(q)A^{-1}(q) k(q,\dot q) - \Lambda(q) \dot J(q) \dot q
\end{equation*}
</div>
<div>
\begin{equation}
\Lambda=(J A^{-1} J^T)^{-1} \label{eq:opinertia}
\end{equation}
</div>
This differential equation $\eqref{eq:opdyn}$ is the operational space dynamics where $h(q, \dot q)$ is the gravity, centrifugal and Coriolis forces in operational space. We have defined a new inertia matrix $\eqref{eq:opinertia}$, the operational space inertia matrix $\Lambda$. The term inside the parentheses of $\eqref{eq:opinertia}$ is invertible because it is positive definite since $A$ is positive definite and $J$ full row rank. This also implies $\Lambda$ is positive definite and therefore a viable inertia matrix.

It is good practice to do [dimensional analysis](https://en.wikipedia.org/wiki/Dimensional_analysis) to check that the units would make sense. A common scenario would be if the joint space inertia has dimension $dim\ A = [M][L]^2$ (mass times length squared) and joint space coordinates are measured in radians and therefore dimensionless, and the operational space operates only in position measured in lengths, then we have $dim\ J=[L]/[1]$, and the dimension of the operational space inertia would be
<div>
\begin{equation*}
dim\ \Lambda = (([M][L]^2)^{-1} ([L]/[1])^2))^{-1} = [M]
\end{equation*}
</div>

### Null space and dynamically consistent generalized inverse
As mentioned at the start there is a subspace of the joint torque space that corresponds to a particular operational force $F$ and one of the torques in this subspace is always $\tau = J^TF$, we therefore have solutions
<div>
\begin{equation}
\tau = \underbrace{J^TF}_{\text{particular solution}} + \underbrace{\tau_{null}}_{\text{null solution}} \label{eq:opdynsol}
\end{equation}
</div>
where $\tau_{null}$ is a joint torque vector such that $F$ is still the only operational force acting on the end effector. 

A parallel can be drawn with linear algebra where if we have a rank deficient matrix $B$, and there is a particular solution $x_p$ to $Bx = b$ then there exists infinite solutions since $B$ has a nullspace. This is because for any (null solution) $x_n$ in the nullspace of $B$ we have $Bx_n = 0$ and therefore $x=x_p+x_n$ are also solutions to the equation.

We want to find a way to construct null solutions to $\eqref{eq:opdynsol}$. If we had a matrix $\tilde{J}$ such that it was a reflexive generalized inverse of $J$
<div>
\begin{equation}
J = J\tilde{J} J \label{eq:geninv}
\end{equation}
</div>
<div>
\begin{equation}
\tilde{J} = \tilde{J} J\bar  J \label{eq:geninvref}
\end{equation}
</div>
and it is a dynamically consistent generalized inverse such that the linear equation
<div>
\begin{equation*}
\tilde{J}^T\tau = F
\end{equation*}
</div>
also holds. Then all solutions $\tau$ to the above equation can be written as 
<div>
\begin{equation*}
\tau = J^TF + \underbrace{(I-J^T\tilde{J}^T)v}_{\tau_{null}}
\end{equation*}
</div>
where $v$ is any torque vector. The second term on the right can express all null solutions. If $J$ was invertible these conditions would be satisfied by $J^{-1}$, however, it is not invertible in our scenario.

We now wish to find the generalized inverse $\tilde{J}$, we know $\tau_{null}$ cannot affect the operational space dynamics and therefore if we rederive the operational space dynamics with the null solution included
<div>
\begin{equation*}
\ddot x = \dot J \dot q + J A^{-1}(J^TF + \tau_{null}-k)
\end{equation*}
</div>
and rearrange the equation and expand $\tau_{null}$ then
<div>
\begin{equation}
\begin{aligned}
\Lambda(q) \ddot x + h(q,\dot q) &= F + \Lambda^{-1} J A^{-1}\tau_{null} \\ 
                                 &= F + \Lambda^{-1} J A^{-1}(I-J^T\tilde{J}^T)v
\end{aligned}
\end{equation}
</div>
the last term needs to be a zero vector for any torque vector $v$. We therefore have

<div>
\begin{equation*}
\Lambda^{-1} J A^{-1}(I-J^T\tilde{J}^T) = 0
\end{equation*}
</div>
which gives the dynamically consistent generalized inverse of $J^T$
<div>
\begin{equation*}
\tilde{J}^T = \Lambda J A^{-1}
\end{equation*}
</div>

This matrix can be applied to $\eqref{eq:jdyn}$ to give
<div>
\begin{equation*}
\tilde{J}^T \tau = \tilde{J}^T (A\ddot q + k)
=\Lambda \ddot x + h = F
\end{equation*}
</div>
in addition, properties $\eqref{eq:geninv}$, $\eqref{eq:geninvref}$ can also be verified to hold for $\tilde{J}$. Also, the coefficient of $k(q,\dot q)$ in $X$ can be replaced by $\tilde{J}^T$.


### Conclusion

We have derived the operational space dynamics and the dynamically consistent generalized inverse. Furthermore, the null space projection matrix $(I-J^T\tilde{J}^T)$ can later be used to find torques that accomplish a lower prioritised second task in the null space of a first task.


### Further reading


### References
https://cs.stanford.edu/groups/manips/publications/pdfs/Khatib_1987_RA.pdf

https://studywolf.wordpress.com/site-index/