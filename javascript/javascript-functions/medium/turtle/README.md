# Turtle

Python comes with a `turtle` module that is a great graphical way to introduce programming to kids. The turtle starts at `(0, 0)` on an x-y plane, and you can move the turtle around and change the direction it's facing. When the turtle moves, it leaves a line trail behind it that appears on a canvas. With these movement APIs, users can draw simple shapes and more complex ones by leveraging loops.

Implement a `Turtle` class in JavaScript that simulates the movement of a turtle. The turtle starts at `(0, 0)` facing north, and it has the following methods:

- `forward(distance)`: Moves the turtle forward by `distance` units in the direction it is facing.
- `backward(distance)`: Moves the turtle backward by `distance` units while facing the same direction.
- `left()`: Rotates the turtle in-place 90 degrees to the left, changing only the direction it is facing.
- `right()`: Rotates the turtle in-place 90 degrees to the right, changing only the direction it is facing.
- `position()`: Returns the coordinates of the turtle as `[x, y]`.

## Example(s)

```ts
const turtle = new Turtle();
turtle.position(); // [0, 0]
turtle.forward(1); // Position: [0, 1]
turtle.backward(1); // Position: [0, 0]
turtle.right(); // Position remains unchanged
turtle.forward(2); // Position: [2, 0] because it moved 2 units to the right.

// Methods can also be chained.
turtle.right().right().forward(5); // Position: [-3, 0] because it turned 180 degrees and moved 5 units forward (toward the left).
```

## Reference(s)

- Python [turtle](https://docs.python.org/3/library/turtle.html)


## Solution(s)

```ts
const DIRECTION = {
  NORTH: 0,
  EAST: 1,
  SOUTH: 2,
  WEST: 3,
};

const PLANE = {
  x: 0,
  y: 1,
};

export default class Turtle {
  #coordinates;
  #direction;

  constructor() {
    this.#coordinates = [0, 0];
    this.#direction = DIRECTION.NORTH;
  }

  /**
   * @param {number} distance Distance to move forward while facing the current direction.
   * @return {Turtle}
   */
  forward(distance: number): Turtle {
    distance =
      this.#direction === DIRECTION.NORTH || this.#direction === DIRECTION.EAST
        ? distance
        : -distance;

    const plane =
      this.#direction === DIRECTION.EAST || this.#direction === DIRECTION.WEST
        ? PLANE.x
        : PLANE.y;

    this.#coordinates[plane] += distance;

    return this;
  }

  /**
   * @param {number} distance Distance to move backward while facing the current direction.
   * @return {Turtle}
   */
  backward(distance: number): Turtle {
    distance =
      this.#direction === DIRECTION.NORTH || this.#direction === DIRECTION.EAST
        ? -distance
        : distance;

    const plane =
      this.#direction === DIRECTION.EAST || this.#direction === DIRECTION.WEST
        ? PLANE.x
        : PLANE.y;

    this.#coordinates[plane] += distance;

    return this;
  }

  /**
   * Turns the turtle left.
   * @return {Turtle}
   */
  left(): Turtle {
    this.#direction =
      this.#direction === DIRECTION.NORTH ? DIRECTION.WEST : --this.#direction;
    return this;
  }

  /**
   * Turns the turtle right.
   * @return {Turtle}
   */
  right(): Turtle {
    this.#direction =
      this.#direction === DIRECTION.WEST ? DIRECTION.NORTH : ++this.#direction;
    return this;
  }

  /**
   * @return {[number, number]} Coordinates [x, y]
   */
  position(): [number, number] {
    return [this.#coordinates[0], this.#coordinates[1]];
  }
}
```
