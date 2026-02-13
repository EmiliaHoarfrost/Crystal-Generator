# Crystal-Generator
WebAssembly crystal generator

---
# WebAssembly Hello World in C

This example shows how to compile a simple C program to WebAssembly using **Emscripten** and run it in the browser. [web:2][web:10]

## 1. C source (`hello.c`)

```c
#include <stdio.h>

int main()
{
    printf("Hello, WebAssembly!\\n");
    return 0;
}
```

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Crystal Lattice Viewer</title>
  <style>
    body { margin: 0; display: flex; height: 100vh; font-family: sans-serif; }
    #controls { width: 260px; padding: 12px; box-sizing: border-box; background: #f4f4f4; }
    #view { flex: 1; }
    label { display: block; margin-top: 6px; font-size: 12px; }
    input, select { width: 100%; box-sizing: border-box; }
    button { margin-top: 10px; width: 100%; }
  </style>
</head>
<body>
  <div id="controls">
    <h3>Unit cell</h3>
    <label>a <input id="a" type="number" step="0.01" value="1.0"></label>
    <label>b <input id="b" type="number" step="0.01" value="1.0"></label>
    <label>c <input id="c" type="number" step="0.01" value="1.0"></label>
    <label>α (deg) <input id="alpha" type="number" step="0.1" value="90"></label>
    <label>β (deg) <input id="beta" type="number" step="0.1" value="90"></label>
    <label>γ (deg) <input id="gamma" type="number" step="0.1" value="90"></label>

    <h3>Bravais lattice</h3>
    <select id="bravais">
      <option value="P_cubic">Cubic P</option>
      <option value="I_cubic">Cubic I</option>
      <option value="F_cubic">Cubic F</option>
      <option value="P_tetragonal">Tetragonal P</option>
      <option value="I_tetragonal">Tetragonal I</option>
      <option value="P_orthorhombic">Orthorhombic P</option>
      <option value="C_orthorhombic">Orthorhombic C</option>
      <option value="I_orthorhombic">Orthorhombic I</option>
      <option value="F_orthorhombic">Orthorhombic F</option>
      <option value="P_monoclinic">Monoclinic P</option>
      <option value="C_monoclinic">Monoclinic C</option>
      <option value="P_triclinic">Triclinic P</option>
      <option value="P_hexagonal">Hexagonal P</option>
      <option value="R_trigonal">Rhombohedral (R)</option>
    </select>

    <button id="update">Update lattice</button>
  </div>
  <div id="view"></div>

  <script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script>
  <script src="https://unpkg.com/three@0.160.0/examples/js/controls/OrbitControls.js"></script>
  <script src="main.js"></script>
</body>
</html>
