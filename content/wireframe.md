+++
title = "Wireframe"
+++

A collection of shaders and materials for high-quality wireframe rendering in Unity.
The package relies on geometry shaders so at the time of release it only worked on Windows and DX10-compatible video cards.
Currenly discontinued.

Features:

- Antialiased lines with adjustable width.
- Lines are calculated in window space and don’t depend on object’s distance from the camera. The farther the object is from the camera, the denser its wireframe. This behavior is the same as for classic wireframe (used, for example, in Unity editor).
- Deferred and forward rendering support.
- Six example materials, including single side, two-sided, solid (wireframe + fill color). Wireframe material can be mixed with any other material by simply assigning two (or more) materials to the same object. This requires at least two draw calls though. For simple wireframe rendering with fill color there is a shader included that requires only one draw call.

{{ img(id="/img/wireframe/screenshot1.png") }}
{{ img(id="/img/wireframe/screenshot2.png") }}
{{ img(id="/img/wireframe/screenshot3.png") }}
