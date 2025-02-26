<script lang="ts">
  import { T } from '@threlte/core'
  import { useGltf, useTexture, useCursor } from '@threlte/extras'
  import type { Mesh, MeshStandardMaterial, Texture } from 'three'
  import { MathUtils } from 'three'
  import { Tween } from 'svelte/motion'
  import { StickPosition, Button } from './types'

  type GLTFResult = {
    nodes: {
      BodyMesh: Mesh
      LeftCover: Mesh
      RightCover: Mesh
      ScreenFrame: Mesh
      joystick_base: Mesh
      joystick_stick_application: Mesh
      joystick_stick: Mesh
      joystick_cap: Mesh
      Main_Button_Enclosure: Mesh
      Main_Button: Mesh
      Screen: Mesh
    }
    materials: {
      ['machine body main']: MeshStandardMaterial
      ['machine body outer']: MeshStandardMaterial
      ['screen frame']: MeshStandardMaterial
      ['joystick base']: MeshStandardMaterial
      ['joystick stick']: MeshStandardMaterial
      ['joystick cap']: MeshStandardMaterial
      Screen: MeshStandardMaterial
    }
  }

  type Props = {
    joystick?: StickPosition
    button?: Button
    screenTexture?: Texture | undefined
    screenClicked?: () => void
  }

  let {
    joystick = StickPosition.Idle,
    button = Button.Idle,
    screenTexture,
    screenClicked
  }: Props = $props()

  const { onPointerEnter, onPointerLeave } = useCursor('pointer')

  const gltf = useGltf<GLTFResult>('/models/ball-game/archade-machine/arcade_machine_own.glb').then(
    (gltf) => {
      Object.entries(gltf.materials).forEach(([name, material]) => {
        const n = name as keyof typeof gltf.materials
        if (n === 'joystick cap') material.envMapIntensity = 1
        else if (n === 'joystick stick') material.envMapIntensity = 1
        else material.envMapIntensity = 0.2
      })
      return gltf
    }
  )
  const scanLinesTexture = useTexture('/models/ball-game/archade-machine/textures/scanlines.png')

  const stickRotation = new Tween(0, {
    duration: 100
  })

  $effect(() => {
    if (joystick == StickPosition.Left) {
      stickRotation.set(-15 * MathUtils.DEG2RAD)
    } else if (joystick == StickPosition.Right) {
      stickRotation.set(15 * MathUtils.DEG2RAD)
    } else {
      stickRotation.set(0)
    }
  })
</script>

{#await gltf then model}
  <!-- Generated by gltfjsx -->
  <T.Group rotation.y={MathUtils.DEG2RAD * 180}>
    <!-- The Main Body -->
    <T.Mesh
      geometry={model.nodes.BodyMesh.geometry}
      material={model.materials['machine body main']}
      position={[0.2755, 0, 0]}
    />
    <T.Mesh
      geometry={model.nodes.LeftCover.geometry}
      material={model.materials['machine body outer']}
      position={[0.3, 1.2099, -0.1307]}
    />
    <T.Mesh
      geometry={model.nodes.RightCover.geometry}
      material={model.materials['machine body outer']}
      position={[-0.3, 1.2099, -0.1307]}
      scale={[-1, 1, 1]}
    />
    <T.Mesh
      geometry={model.nodes.ScreenFrame.geometry}
      material={model.materials['screen frame']}
      position={[0.2755, 0.0633, 0.0346]}
    />

    <!-- Joystick -->
    <T.Mesh
      geometry={model.nodes.joystick_base.geometry}
      material={model.materials['joystick base']}
      position={[0.1336, 0.9611, -0.1976]}
      rotation={[-0.1939, 0, 0]}
    />
    <T.Mesh
      geometry={model.nodes.joystick_stick_application.geometry}
      material={model.materials['joystick base']}
      position={[0.1336, 0.9653, -0.1984]}
      rotation={[-0.1939, 0, stickRotation.current]}
    >
      <T.Mesh
        geometry={model.nodes.joystick_stick.geometry}
        material={model.materials['joystick stick']}
        position={[0, -0.0145, 0.0001]}
      >
        <T.Mesh
          geometry={model.nodes.joystick_cap.geometry}
          material={model.materials['joystick cap']}
          position={[-0.0001, 0.1126, -0.0005]}
          material.envMapIntensity={0.5}
        />
      </T.Mesh>
    </T.Mesh>

    <!-- The Button -->
    <T.Mesh
      geometry={model.nodes.Main_Button_Enclosure.geometry}
      material={model.materials['joystick base']}
      position={[-0.1143, 0.9795, -0.0933]}
      rotation={[-0.1801, 0, 0]}
      scale={0.9409}
    >
      <T.Mesh
        geometry={model.nodes.Main_Button.geometry}
        material={model.materials['joystick cap']}
        position={[0.0001, 0.007 + (button == Button.Pressed ? -0.003 : 0), -0.0003]}
        rotation={[0.192, 0, 0]}
        scale={0.724}
      />
    </T.Mesh>

    <!-- The screen itself gets a special treatment -->
    <T.Mesh
      geometry={model.nodes.Screen.geometry}
      position={[0, 1.3774, 0.1447]}
      scale={1.0055}
      onpointerenter={onPointerEnter}
      onpointerleave={onPointerLeave}
      onclick={() => {
        screenClicked?.()
      }}
    >
      {#await scanLinesTexture then texture}
        {#if screenTexture}
          <T.MeshStandardMaterial
            metalness={0.9}
            roughness={0.2}
            map={screenTexture}
            metalnessMap={texture}
          />
        {:else}
          <T.MeshStandardMaterial
            metalness={0.9}
            roughness={0.2}
            color="#141414"
            metalnessMap={texture}
          />
        {/if}
      {/await}
    </T.Mesh>
  </T.Group>
{/await}
