---
topic: unity
status: wip
---

# player control

## three movement approaches

Pick one. Don't mix.

**Transform translate.** `transform.Translate(dir * speed * Time.deltaTime)`. No physics. Walks through walls unless you bolt on a collider check. Fine for prototypes and UI.

**Rigidbody.** Physics-driven. Set `rb.linearVelocity` or call `rb.AddForce()` in `FixedUpdate`. Collisions, friction, gravity come free. Use it for anything that has to react to the world.

**CharacterController.** Capsule controller built for humanoids. `controller.Move(motion)`. Ignores forces, so push it yourself. Handles slopes, steps, ground contact. Most third-person and FPS controllers run on this.

## input

Two systems.

**Legacy Input Manager.** `Input.GetAxis("Horizontal")`, `Input.GetKeyDown(KeyCode.Space)`. Simple, global, falls apart the moment you want rebinding.

**New Input System package.** Action assets, per-device bindings, callbacks through a `PlayerInput` component or a generated C# class. Use this for anything you ship.

Read input in `Update`. Cache the values. Apply motion where it belongs: `FixedUpdate` for Rigidbody, `Update` for Transform and CharacterController.

## ground check

Needed for jump, slide, animation state.

- `Physics.CheckSphere(feetPos, radius, groundMask)`. Cheap and good enough.
- Raycast straight down from the feet, length is a small epsilon.
- CharacterController has `controller.isGrounded`. Flaky on slopes, double-check with a raycast.

Set a `LayerMask` for ground in the Inspector. Don't use tag strings.

## jump

Different per movement type.

**Rigidbody.** `rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse)` when grounded.

**CharacterController.** Track vertical velocity yourself. On jump set `velY = sqrt(2 * jumpHeight * gravity)`. Apply gravity each frame: `velY -= gravity * Time.deltaTime`. Reset to a small negative value when grounded so the controller stays stuck to the floor.

## mouse look / camera

First-person setup.

- Read `Input.GetAxis("Mouse X")` and `"Mouse Y"`, or the new Input System Delta.
- Yaw on the player body. Rotate around Y.
- Pitch on the camera child. Rotate around X. Clamp to `[-89, 89]` so you can't flip.
- In `Awake`: `Cursor.lockState = CursorLockMode.Locked; Cursor.visible = false;`.

## frame vs physics step

Recap, because it bites.

- `Update` runs every render frame. Variable rate.
- `FixedUpdate` runs at a fixed physics tick. Default 50 Hz.
- Read input in `Update`. Apply physics in `FixedUpdate`.
- Multiply Transform and CharacterController moves by `Time.deltaTime`.
- Don't multiply Rigidbody force by `Time.deltaTime`. `AddForce` is already per-step.

## decouple input from movement

Controller takes a `Vector2 moveInput` from outside. Set by an input handler, AI, or replay. Same controller works for player and AI. Don't read `Input.*` inside the movement script.

## minimal example

Rigidbody-based third-person mover with jump.

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class PlayerController : MonoBehaviour
{
    [SerializeField] private float speed = 6f;
    [SerializeField] private float jumpForce = 5f;
    [SerializeField] private Transform feet;
    [SerializeField] private float groundRadius = 0.2f;
    [SerializeField] private LayerMask groundMask;

    private Rigidbody rb;
    private Vector2 input;
    private bool jumpQueued;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true;
    }

    void Update()
    {
        input.x = Input.GetAxis("Horizontal");
        input.y = Input.GetAxis("Vertical");
        if (Input.GetButtonDown("Jump")) jumpQueued = true;
    }

    void FixedUpdate()
    {
        Vector3 move = transform.right * input.x + transform.forward * input.y;
        Vector3 vel = rb.linearVelocity;
        vel.x = move.x * speed;
        vel.z = move.z * speed;

        if (jumpQueued && IsGrounded())
        {
            vel.y = jumpForce;
        }
        jumpQueued = false;

        rb.linearVelocity = vel;
    }

    private bool IsGrounded()
    {
        return Physics.CheckSphere(feet.position, groundRadius, groundMask);
    }
}
```
