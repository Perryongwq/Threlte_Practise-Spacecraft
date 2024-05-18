<script>
  import { T, useFrame, useThrelte } from '@threlte/core'
  import { ContactShadows, Float, Grid, OrbitControls } from '@threlte/extras'
  import StarWarShip from './models/star_wars_battlefront_-_tie_fighter.svelte'
  import {MeshBasicMaterial, PlaneGeometry, Raycaster, SphereGeometry, Vector2, Vector3} from 'three'
  import {onMount} from 'svelte'

  let planeRef;
  let sphereRef;

  const { scene, camera, renderer } = useThrelte();
  let intersectionPoint;
  let translY = 0;
  let translAcceleration = 0;
  let angleZ = 0;
  let angleAcceleration = 0;

	useFrame(({ scene }) => {
		if (intersectionPoint) {
			const targetY = intersectionPoint?.y || 0;
			translAcceleration += (targetY - translY) * 0.002; // stiffness
			translAcceleration *= 0.95; // damping
			translY += translAcceleration;

			const dir = intersectionPoint.clone().sub(new Vector3(0, translY, 0)).normalize();
			const dirCos = dir.dot(new Vector3(0, 1, 0));
			const angle = Math.acos(dirCos) - Math.PI * 0.5;
			angleAcceleration += (angle - angleZ) * 0.01; // stiffness
			angleAcceleration *= 0.85; // damping
			angleZ += angleAcceleration;
		}
	});

	onMount(() => {
		const raycaster = new Raycaster();
		const pointer = new Vector2();

		function onPointerMove(event) {
			pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
			pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

			raycaster.setFromCamera(pointer, $camera);
			const intersects = raycaster.intersectObject(planeRef);
			intersectionPoint = intersects[0]?.point;

			if (intersectionPoint) {
				// this prevents the spring motion to be different while the pointer
				// spans the x axis
				intersectionPoint.x = -3;
				sphereRef.position.copy(intersectionPoint);
			}
		}

		window.addEventListener('pointermove', onPointerMove);
		return () => {
			window.removeEventListener('pointermove', onPointerMove);
		};
	});
</script>

<T.PerspectiveCamera makeDefault position ={[-5,6,10]} fov ={25}>
<OrbitControls enableDamping target={[0,0,0]} />
</T.PerspectiveCamera>

<T.DirectionalLight intensity={1.8} position={[0,10,0]} castShadow  shadow.bias={-0.0001}/>
<T.AmbientLight intensity={0.2} />

<Grid 
  position.y ={-0.001}
  cellColor='#ffffff'
  sectionColor='#ffffff'
  sectionThickness={0}
  fadeDirection={25}
  cellSize={2}
/>

<StarWarShip position = {[0, translY, 0]} rotation= {[angleZ, 0, angleZ, 'ZXY']}/>

<T.Mesh renderOrder={2} bind:ref={planeRef} visible={false}>
	<T.PlaneGeometry args={[20, 20]} />
	<T.MeshBasicMaterial color={[1, 0, 1]} transparent opacity={0.25} />
</T.Mesh>

<T.Mesh position={[1, 2, 0]} bind:ref={sphereRef} visible={false}>
	<T.SphereGeometry args={[0.1, 20, 20]} />
	<T.MeshBasicMaterial color={[1, 0, 0]} />
</T.Mesh>