---
title: Triangle Intersection
date: 2020-11-22 19:20:47
tags: [Ray Trace, OpenGL]
categories: [Graphics]
---

$\newcommand{\Vc}[3]{\begin{bmatrix}
  #1 \\\ #2 \\\ #3 \\\
\end{bmatrix} }$

$\newcommand{\D}[2]{
  #1 \cdot #2
}$

$\newcommand{\Frac}[2]{
  #1 \over #2
}$

# Triangle intersection

Last time, we have talked about the method to find an intersection from a ray to a sphere, which we simply find the common root(s) by substituting the x,y,z in sphere equation with our ray equation. This time, however, is a bit harder than just simply solving the quadratic formula when determining whether there is an intersection between a ray and a triangle.

![](triangle_intersection.png)

According to the picture, we can define an equation for the ray:

\begin{equation}
  P(t) = p + td\label{eq1}\\
\end{equation}

p = origin of the point, t = time, d = a vector indicating direction, and we have a triangle with three vertex position A, B, C.

Now, the question comes down to how we are able to find intersection point Q?

There are two major steps we need to take to reach our goal:
 - Since there is exactly one plane that contains any three vertex positions, and this can be determined with the normal vector computed from these three vertex positions. Thus, our first step is to check whether the ray hits the supporting plane of this triangle.
 - Then, we also need to check whether Q is inside the triangle ABC since a triangle is bounded.

## Plane Intersection

Our plane equation is:

\begin{equation}
  ax + by + cz + d = 0\label{eq2}
\end{equation}

Which can be expressed as two column vectors dot product:


\begin{equation}
  \Vc{a}{b}{c}
  \cdot
  \Vc{x}{y}{z}
  = k \label{eq3}
\end{equation}


Let's denote $n = \Vc{a}{b}{c}$ and $x = \Vc{x}{y}{z}$, now we can substitute variable $x$ with our ray equation for $P(t)$ because essentially our ray equation will yield a point as a vector like $\Vc{P_x(t)}{P_y(t)}{P_z(t)}$.

We will get an equation like the following:


\begin{align*}
  \D{n}{(p + td)} = k
\end{align*}

\begin{align*}
  \D{n}{p} + \D{tn}{d} = k
\end{align*}

\begin{equation}
  t = {\Frac{k - \D{n}{p}}{\D{n}{d}}}\label{eq4}
\end{equation}

Note that if $\D{n}{d}$ is 0, meaning ray and plane are parallel. Thus, We need to check this before plugging the numbers.

In this equation, we have two unknows k and vector n. How do we find them?

Well for n, as its symbol suggests, it is actually the normal vector of the plane. Also, remember our equation for calculating a normal vector with two vectors that are not parallel, so we can solve for our normal vector as following:

\begin{equation}
  n = {\Frac{(B - A) \times (C - A)}{||(B - A) \times (C - A)||}}\label{eq5}
\end{equation}

Yeah, we have another unknown variable solved and usually we want to normalize it for model shading purpose. Finally, solving k is just to plug in one of three points of that triangle:

\begin{align*}
  k = \D{n}{A}
\end{align*}


\begin{equation}
  k = \D{\Vc{a}{b}{c}}{\Vc{A_x}{A_y}{A_z}}
\end{equation}

Now, we have everything we need to calculate t, and we can also calculate point of intersection of the ray and plane. Let's call that Q.

## Triangle inside-outside testing

![](inside_outside.png)

The way to test whether the point lies inside or outside the triangle is using the sign of the normal vector computed from the vector of each side of triangle and the vector composed by each point of the triangle minus point Q. Note that if result is 0, it means point Q lies on one of the three sides:

\begin{equation}
  \D{[(B - A) \times (Q - A)]}{n} \ge 0
\end{equation}

\begin{equation}
  \D{[(C - B) \times (Q - B)]}{n} \ge 0
\end{equation}

\begin{equation}
  \D{[(A - C) \times (Q - C)]}{n} \ge 0
\end{equation}

If any one of these tests fail, the point is not inside the triangle; otherwise, it is inside the triangle!

## Code Example

{% codeblock Triangle::IntersectLocal lang:c++ %}
bool TriangleFace::IntersectLocal(const Ray &r, Intersection &i)
{
  /*
  glm is math library that resembles glsl shader language.
  Main info for Ray:
  struct Ray {
    glm::dvec3 position;
    glm::dvec3 direction;
  };
  Main info for Intersection:
  struct Intersection {
    double t;
    glm::vec3 normal;
  };

  A, B, C are member vertices that are glm::dvec3 storing position information
  */

  // Calculate the plane normal ((B - A) x (C - A))
  glm::dvec3 AB = {b.x - a.x, b.y - a.y, b.z - a.z};
  glm::dvec3 AC = {c.x - a.x, c.y - a.y, c.z - a.z};
  glm::dvec3 plane_normal = glm::cross(AB, AC);
  plane_normal = glm::normalize(plane_normal);

  glm::dvec3 d = r.direction;
  if (abs(glm::dot(plane_normal, d)) < NORMAL_EPSILON) { // ray is parallel, no intersection
     return false;
  }
  glm::dvec3 p = r.position;

  // Find k using A
  double k = glm::dot(plane_normal, a);

  // Get the t intersect value
  i.t = (k - glm::dot(plane_normal, p)) / glm::dot(plane_normal, d);

  // This is to check that t is not intersecting with itself again because of double and floating precision issue
  if (i.t < RAY_EPSILON) {
     return false;
  }

  // Intersection point on the plane
  glm::dvec3 Q = r.at(i.t);

  // Check if Q isn't within the triangle ABC
  if (glm::dot(glm::cross((b - a), (Q - a)), plane_normal) < EDGE_EPSILON ||
        glm::dot(glm::cross((c - b), (Q - b)), plane_normal) < EDGE_EPSILON ||
        glm::dot(glm::cross((a - c), (Q - c)), plane_normal) < EDGE_EPSILON) {
     return false;
  }

  return true;
}

{% endcodeblock %}

# Reference:
- Fundamentals Of Computer Graphics - Peter Shirley, Steve Marschner
