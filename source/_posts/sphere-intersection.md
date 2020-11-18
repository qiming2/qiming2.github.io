---
title: Sphere Intersection
date: 2020-11-11 19:25:49
tags: [Ray Trace, OpenGL]
categories: [Graphics]
---

# Sphere Intersection

Say if we have a ray:
$$\begin{equation}
P(t) = p + td\label{eq1}\\
\end{equation}$$

p = origin of the point, t = time, d = a vector indicating direction, and we have a sphere with center at point o\\((x_1, y_1, z_1)\\)

$$\begin{equation}
(x - x_1)^2 + (y - y_1)^2 + (z - z_1)^2 - R^2 = 0\label{eq2}\\
\end{equation}$$

Now, we know a ray will intersect with a sphere if we can find a point through \\(P(t)\\) that can satisfy the sphere equation. Thus the equation can be written as:

$$(x_p - x_1)^2 + (y_p - y_1)^2 + (z_p - z_1)^2 - R^2 = 0\tag{3}\label{eq3}$$

Then it can also be rewritten as the following:

$$(p + td - o) \bullet (p + td - o) - R^2 = 0\tag{4}\label{eq4}$$

Finally, we can rearrange all the terms to get a nice quadratic formula:

$$(d \cdot d)t^2 + 2d\cdot (p - o)t + (p - o) \cdot (p - o) - R^2 = 0\tag{5}\label{eq5}$$

According to our quadratic formula, we can get two roots for t:

$$t = {-d \cdot (p - o) \pm \sqrt{(d \cdot (p - o))^2 - (d \cdot d)((p - o) \cdot (p - o) - R^2)}\over (d \cdot d)}\tag{6}\label{eq6}$$

Finally, we can find the intersection by plugging t back into \\(\eqref{eq1}\\) and find a point \\(a\\). Also, the unit normal vector can be calculated easily by \\((a - o)/R\\)

Below code section is a sphere intersection function in C++ that assumes origin of sphere is at (0, 0, 0) with a radius 0.5:

{% codeblock Sphere::IntersectLocal lang:c++ %}

bool Sphere::IntersectLocal(const Ray &r, Intersection &i)
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
    */
    glm::dvec3 P = r.position;
    glm::dvec3 D = r.direction;

    double a = glm::dot(D, D);
    double b = 2 * glm::dot(D, P);
    double c = glm::dot(P, P) - (0.5 * 0.5);

    double discriminant = b * b - (4 * a * c);

    // Since there could be floating point precision issue,
    // we need to check whether t is greater than an epsilon like 0.001
    if (discriminant < RAY_EPSILON) {
        return false;
    }
    // Set the t-value for the intersection
    double t1 = (-b + sqrt(discriminant)) / (2 * a);
    double t2 = (-b - sqrt(discriminant)) / (2 * a);

    // Check whether there are real roots
    if (t2 < t1 && t2 >= RAY_EPSILON) { // tangent ray
        i.t = t2;
    } else if (t1 >= RAY_EPSILON) {
        i.t = t1;
    } else {
        return false;
    }

    // Set the normal for the intersection point
    // and usually we want to normalize it, so we are consistent
    glm::dvec3 intersect_point = r.at(i.t);
    i.normal = glm::normalize(intersect_point);

    return true;
}
{% endcodeblock %}

# Reference:
- Fundamentals Of Computer Graphics - Peter Shirley, Steve Marschner
