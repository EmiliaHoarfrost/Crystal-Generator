<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Crystal Lattice Viewer - Minerals</title>
  <style>
    body { margin:0; display:flex; height:100vh; font-family:sans-serif; }
    #controls { width:260px; padding:12px; background:#f4f4f4; overflow-y:auto; }
    #view { flex:1; }
    #About { width:220px; padding:12px; background:#eaeaea; overflow-y:auto; }
    label, select, input, button { display:block; width:100%; margin-top:6px; box-sizing:border-box; font-size:12px; }
    button { margin-top:10px; }
    h3 { margin-top:12px; margin-bottom:6px; }
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

    <h3>Common Minerals</h3>
    <select id="mineralSelect">
      <option value="">-- Select Mineral --</option>
      <option value="gold">Gold (Au)</option>
      <option value="silver">Silver (Ag)</option>
      <option value="copper">Copper (Cu)</option>
      <option value="diamond">Diamond (C)</option>
      <option value="halite">Halite (NaCl)</option>
      <option value="graphite">Graphite (C)</option>
    </select>

    <h3>Atom Size</h3>
      <label>
      Radius: <input id="atomRadius" type="number" step="0.01" value="0.08" min="0.01" max="1.0">
      </label>

  </div>

  <div id="view"></div>

  <div id="About">
    <h3>About</h3>
    <label>
      This lattice viewer shows atoms as spheres.
      Adjust unit cell parameters or select a mineral to explore different crystal structures.
      Metals have metallic shading.
    </label>
  </div>

  <script type="module">
    import * as THREE from "https://unpkg.com/three@0.160.0/build/three.module.js?module";
    import { OrbitControls } from "https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js?module";

    const viewDiv = document.getElementById('view');

    const renderer = new THREE.WebGLRenderer({ antialias:true });
    renderer.setPixelRatio(window.devicePixelRatio);
    viewDiv.appendChild(renderer.domElement);

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xffffff);

    const camera = new THREE.PerspectiveCamera(45,1,0.1,1000);
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    scene.add(new THREE.AxesHelper(2));

    const light = new THREE.DirectionalLight(0xffffff, 0.8);
    light.position.set(5,5,5);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0xffffff,0.4));

    window.addEventListener('resize', ()=>{ 
      const w=viewDiv.clientWidth, h=viewDiv.clientHeight;
      renderer.setSize(w,h);
      camera.aspect=w/h;
      camera.updateProjectionMatrix();
    });

    // Global atom material to allow mineral updates
    const atomMaterial = new THREE.MeshStandardMaterial({
      color: 0x0077ff, metalness:0, roughness:0.8
    });

    const mineralData = {
      gold:{a:4.078,b:4.078,c:4.078,alpha:90,beta:90,gamma:90,bravais:"F_cubic",color:0xffd700,metal:true},
      silver:{a:4.085,b:4.085,c:4.085,alpha:90,beta:90,gamma:90,bravais:"F_cubic",color:0xc0c0c0,metal:true},
      copper:{a:3.615,b:3.615,c:3.615,alpha:90,beta:90,gamma:90,bravais:"F_cubic",color:0xb87333,metal:true},
      diamond:{a:3.567,b:3.567,c:3.567,alpha:90,beta:90,gamma:90,bravais:"F_cubic",color:0xffffff,metal:false},
      halite:{a:5.640,b:5.640,c:5.640,alpha:90,beta:90,gamma:90,bravais:"F_cubic",color:0xadd8e6,metal:false},
      graphite:{a:2.461,b:2.461,c:6.708,alpha:90,beta:90,gamma:120,bravais:"P_hexagonal",color:0x555555,metal:false}
    };

    document.getElementById('mineralSelect').addEventListener('change', e=>{
      const m=mineralData[e.target.value];
      if(!m) return;

      document.getElementById('a').value=m.a;
      document.getElementById('b').value=m.b;
      document.getElementById('c').value=m.c;
      document.getElementById('alpha').value=m.alpha;
      document.getElementById('beta').value=m.beta;
      document.getElementById('gamma').value=m.gamma;
      document.getElementById('bravais').value=m.bravais;

      atomMaterial.color.set(m.color);
      atomMaterial.metalness=m.metal?1:0;
      atomMaterial.roughness=m.metal?0.2:0.8;

      buildLattice(getParams());
    });

    function basisForBravais(type){
      switch(type){
        case 'I_cubic': case 'I_tetragonal': case 'I_orthorhombic': return [[0,0,0],[0.5,0.5,0.5]];
        case 'F_cubic': case 'F_orthorhombic': return [[0,0,0],[0,0.5,0.5],[0.5,0,0.5],[0.5,0.5,0]];
        case 'C_orthorhombic': case 'C_monoclinic': return [[0,0,0],[0.5,0.5,0]];
        case 'R_trigonal': return [[0,0,0],[2/3,1/3,1/3],[1/3,2/3,2/3]];
        default: return [[0,0,0]];
      }
    }

    function clearSceneObjects(){
      const toRemove=[];
      scene.traverse(obj=>{if(obj.userData?.isLattice) toRemove.push(obj)});
      toRemove.forEach(obj=>{scene.remove(obj); obj.geometry?.dispose(); obj.material?.dispose();});
    }

    function addUnitCellPlanes(aVec,bVec,cVec){
      const planeMat=new THREE.MeshStandardMaterial({color:0xffaa00,side:THREE.DoubleSide,transparent:true,opacity:0.25});
      const corners=[
        new THREE.Vector3(0,0,0),
        aVec.clone(),
        bVec.clone(),
        cVec.clone(),
        aVec.clone().add(bVec),
        aVec.clone().add(cVec),
        bVec.clone().add(cVec),
        aVec.clone().add(bVec).add(cVec)
      ];
      const faces=[[0,1,4,2],[0,1,5,3],[0,2,6,3],[7,5,1,4],[7,6,2,4],[7,5,3,6]];
      faces.forEach(idx=>{
        const geom=new THREE.BufferGeometry();
        const vertices=new Float32Array([
          ...corners[idx[0]].toArray(),
          ...corners[idx[1]].toArray(),
          ...corners[idx[2]].toArray(),
          ...corners[idx[2]].toArray(),
          ...corners[idx[3]].toArray(),
          ...corners[idx[0]].toArray()
        ]);
        geom.setAttribute('position',new THREE.BufferAttribute(vertices,3));
        geom.computeVertexNormals();
        const mesh=new THREE.Mesh(geom,planeMat);
        mesh.userData.isLattice=true;
        scene.add(mesh);
      });
    }

    function addLatticeLines(positions){
      const material=new THREE.LineBasicMaterial({color:0x999999,opacity:0.5,transparent:true});
      const pointMap=new Map();
      positions.forEach(p=>{pointMap.set(`${Math.round(p.x*1000)},${Math.round(p.y*1000)},${Math.round(p.z*1000)}`,p.clone())});
      const neighborOffsets=[new THREE.Vector3(1,0,0),new THREE.Vector3(0,1,0),new THREE.Vector3(0,0,1)];
      positions.forEach(p=>{neighborOffsets.forEach(offset=>{
        const neighbor=p.clone().add(offset);
        const key=`${Math.round(neighbor.x*1000)},${Math.round(neighbor.y*1000)},${Math.round(neighbor.z*1000)}`;
        if(pointMap.has(key)){
          const geom=new THREE.BufferGeometry().setFromPoints([p,pointMap.get(key)]);
          const line=new THREE.Line(geom,material);
          line.userData.isLattice=true;
          scene.add(line);
        }
      })});
    }

    function buildLattice(params){
      clearSceneObjects();
      const a=params.a,b=params.b,c=params.c;
      const alpha=params.alpha*Math.PI/180,beta=params.beta*Math.PI/180,gamma=params.gamma*Math.PI/180;
      const basis=basisForBravais(params.bravais);
      const nx=3,ny=3,nz=3;
      const aVec=new THREE.Vector3(a,0,0);
      const bVec=new THREE.Vector3(b*Math.cos(gamma),b*Math.sin(gamma),0);
      const cx=c*Math.cos(beta);
      const cy=c*(Math.cos(alpha)-Math.cos(beta)*Math.cos(gamma))/(Math.sin(gamma||1e-6));
      const cz=Math.sqrt(Math.max(0,c*c-cx*cx-cy*cy));
      const cVec=new THREE.Vector3(cx,cy,cz);

      addUnitCellPlanes(aVec,bVec,cVec);

      const radius = parseFloat(document.getElementById('atomRadius').value) || 0.08;
      const atomGeom = new THREE.SphereGeometry(radius, 20, 20);

      
      const positions=[];
      for(let i=0;i<nx;i++) for(let j=0;j<ny;j++) for(let k=0;k<nz;k++){
        const cellOrigin=new THREE.Vector3().addScaledVector(aVec,i).addScaledVector(bVec,j).addScaledVector(cVec,k);
        basis.forEach(bfrac=>{
          const pos=new THREE.Vector3().copy(cellOrigin).addScaledVector(aVec,bfrac[0]).addScaledVector(bVec,bfrac[1]).addScaledVector(cVec,bfrac[2]);
          positions.push(pos);
          const atom=new THREE.Mesh(atomGeom,atomMaterial);
          atom.position.copy(pos);
          atom.userData.isLattice=true;
          scene.add(atom);
        });
      }

      const center=new THREE.Vector3();
      positions.forEach(p=>center.add(p));
      center.multiplyScalar(1/positions.length);

      addLatticeLines(positions);

      scene.traverse(obj=>{if(obj.userData?.isLattice) obj.position.sub(center)});

      let maxDist=0;
      positions.forEach(p=>maxDist=Math.max(maxDist,p.distanceTo(center)));

      const dist=maxDist*3+0.5;
      controls.target.set(0,0,0);
      camera.position.set(dist,dist,dist);
      camera.lookAt(0,0,0);
      controls.update();
    }

    function getParams(){ return {
      a:parseFloat(document.getElementById('a').value)||1,
      b:parseFloat(document.getElementById('b').value)||1,
      c:parseFloat(document.getElementById('c').value)||1,
      alpha:parseFloat(document.getElementById('alpha').value)||90,
      beta:parseFloat(document.getElementById('beta').value)||90,
      gamma:parseFloat(document.getElementById('gamma').value)||90,
      bravais:document.getElementById('bravais').value
    }; }

    document.getElementById('update').addEventListener('click',()=>buildLattice(getParams()));

    window.addEventListener('resize', ()=>{onResize();});
    function onResize(){const w=viewDiv.clientWidth,h=viewDiv.clientHeight;renderer.setSize(w,h);camera.aspect=w/h;camera.updateProjectionMatrix();}
    
    buildLattice(getParams());
    document.getElementById('atomRadius').addEventListener('input', () => {
    buildLattice(getParams());
});

    function animate(){requestAnimationFrame(animate);controls.update();renderer.render(scene,camera);}
    animate();
  </script>
</body>
</html>
