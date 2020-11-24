---
title: Baycentric Coordinates
date: 2020-11-23 15:43:36
tags: [OpenGL, Graphics]
---
$\newcommand{\Vc}[3]{\begin{bmatrix}
#1 \\\ #2 \\\ #3 \\\
\end{bmatrix} }$

$\newcommand{\Frac}[2]{
  #1 \over #2
}$

# Baycentric Coordinates

In OpenGL, vertex usually has many attributes like normal vector, uv coordinates, position vector and even color value, and also models are almost always drawn from a bunch of triangles that define each small piece of surface on those models. However, we can only set values to those vertices of each triangles but not those vertices that lie inside the triangles. How do find those values? This is where baycentric coordinates come into play.

Baycentric coordinates essentially helps us to interpolate the value, like normal vector, uv coordinate, of a point that lies inside a triangle with vertex ABC and has a position = $\Vc{a}{b}{c}$.

![](triangle_intersection.png)

According to the picture now if Q lies inside the triangle, position Q can always be expressed as some proportions of each position vector of each vertex ABC:

\begin{equation}
  Q = {\alpha}A + {\beta}B + {\gamma}C \label{eq1}
\end{equation}

$(\alpha, \beta, \gamma)$ are the baycentric coordinates we are looking for. The baycentric coordinates are proportional to the area of the subtriangles formed by each side with point Q like the following:

\begin{equation}
  \alpha = {\Frac{AreaQBC}{AreaABC}}\label{eq2}
\end{equation}

\begin{equation}
  \beta = {\Frac{AreaQAC}{AreaABC}}\label{eq3}
\end{equation}

\begin{equation}
  \gamma = {\Frac{AreaQAB}{AreaABC}}\label{eq4}
\end{equation}

To calculate the baycentric coordinates we need to calculate the areas of all the subtriangles as well area of the triangle ABC. And this is pretty easy because of normal vector! Remember that half of the magnitude of a normal vector for one triangle is the area of that triangle:

\begin{equation}
  AreaABC = {||(B - A) \times (C - A)|| \over 2} \label{eq5}
\end{equation}

Thus we can apply the same strategy to those three subtriangles (we can get rid of division since they will be canceled anyways):

\begin{equation}
  \alpha = {\Frac{||(C - B) \times (Q - B)||}{||(B - A) \times (C - A)||}}\label{eq6}
\end{equation}

\begin{equation}
  \beta = {\Frac{||(A - C) \times (Q - C)||}{||(B - A) \times (C - A)||}}\label{eq7}
\end{equation}

\begin{equation}
  \gamma = {\Frac{||(B - A) \times (Q - A)||}{||(B - A) \times (C - A)||}}\label{eq8}
\end{equation}

Once we have our baycentric coordinates, we can easily calculate unit normal vector:

\begin{equation}
  N_Q = {\alpha N_A + \beta N_B + \gamma N_C \over ||\alpha N_A + \beta N_B + \gamma N_C||}\label{eq9}
\end{equation}

As well as uv coordinate, but we don't normalize them; otherwise, you would get strange effect unless that is exactly what you want ^ ^!

\begin{equation}
  UV_Q = \alpha UV_A + \beta UV_B + \gamma UV_C \label{eq10}
\end{equation}

## Code Example:

{% codeblock Triangle::Baycentric lang:c++ %}

void TriangleFace::Baycentric(Vertex &Q)
{
  /*
  struct Vertex {
    glm::dvec3 p; // position
    glm::dvec3 n; // normal vector (normalized)
    glm::dvec2 uv; // uv coordinate
  }

  glm is math library that resembles glsl shader language.
  a, b, c are member vertices that are glm::dvec3 storing position information for vertex ABC
  a_n, b_n, c_n are normal vectors (normalized) for vertex ABC
  a_uv, b_uv, c_uv are uv coodinates for vertex ABC
  */

  // Get the barycentric coordinate constants
   double alpha = glm::length(glm::cross(c - b, Q.p - b)) / glm::length(glm::cross(b - a, c - a));
   double beta = glm::length(glm::cross(a - c, Q.p - c)) / glm::length(glm::cross(b - a, c - a));
   double gamma = glm::length(glm::cross(b - a, Q.p - a)) / glm::length(glm::cross(b - a, c - a));

   // Calculate the normal at point Q

   Q.normal = glm::normalize((alpha * a_n) + (beta * b_n) + (gamma * c_n));


   // Calculate the uv coordinates at point Q
   glm::vec2 weighted_a_uv = {a_uv.x * alpha, a_uv.y * alpha};
   glm::vec2 weighted_b_uv = {b_uv.x * beta, b_uv.y * beta};
   glm::vec2 weighted_c_uv = {c_uv.x * gamma, c_uv.y * gamma};
   Q.uv = weighted_a_uv + weighted_b_uv + weighted_c_uv;
}

{% endcodeblock %}

# Reference:
- Fundamentals Of Computer Graphics - Peter Shirley, Steve Marschner
