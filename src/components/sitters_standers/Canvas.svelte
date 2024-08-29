<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let data;

	let circles = [];
	const dotSize = 5; // Smaller size for Person circles
	export let selectedVar;
	let new_selectedVar = "";
	let multiplier = 0.03;

	const sketch = (p) => {
		p.setup = () => {
			w = p.windowWidth;
			h = p.windowHeight - 6;
			p.createCanvas(w, h);

			for (let i = 0; i < data.length; i++) {
				let circle = new Circle(data[i], i);
				circles.push(circle);
			}
		};

		p.draw = () => {
			p.clear();
			p.background(240, 240, 240);

			for (let i = 0; i < circles.length; i++) {
				for (let j = i + 1; j < circles.length; j++) {
					circles[i].collide(circles[j]);
				}
				circles[i].update();
				circles[i].display();
			}

			for (let i = 0; i < circles.length; i++) {
				if (p.dist(p.mouseX, p.mouseY, circles[i].center.x, circles[i].center.y) < circles[i].radius*1.1) {
					circles[i].displayText();	
				}				
			}

			if (selectedVar != new_selectedVar) {
				for (let i = 0; i < circles.length; i++) {
					circles[i].updateGroup();
				}
				new_selectedVar = selectedVar;
			}
		};

		let resizeTimeout;

		p.windowResized = () => {
			w = p.windowWidth;
			h = p.windowHeight - 6;
			p.resizeCanvas(w, h);

			clearTimeout(resizeTimeout);

			resizeTimeout = setTimeout(() => {
				for (let i = 0; i < circles.length; i++) {
					circles[i].updateGroup();
				}
			}, 300);
		};

		class Circle {
			constructor(obj, index) {
				this.obj = obj;
				this.index = index;
				this.radius = Math.max(Math.sqrt(obj.TOT_EMP) * multiplier, dotSize); // Ensure radius is not too small
				if (this.radius < dotSize*1.5) {
					this.radius = dotSize*1.5;
				}
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					p.random(this.radius, h - this.radius)
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.maxSpeed = 10;
				this.maxForce = 10;
				this.varPct = 0;
				this.peoplePositions = this.calculatePeoplePositions(obj.dots);
			}

			calculatePeoplePositions(totalPeople) {
				let positions = [];
				let centerX = this.center.x;
				let centerY = this.center.y;
				let angle = 0;
				let radiusIncrement = dotSize * 1.2; // Adjust this for packing density

				for (let i = 0; i < totalPeople; i++) {
					let x, y;

					if (i === 0) {
						// First ellipse at the center
						x = centerX;
						y = centerY;
					} else {
						// Calculate the position for the next ellipse
						let currentRadius = Math.floor(Math.sqrt(i)) * radiusIncrement;
						angle = (i % Math.floor(Math.sqrt(i))) * p.TWO_PI / Math.floor(Math.sqrt(i));
						x = centerX + currentRadius * p.cos(angle);
						y = centerY + currentRadius * p.sin(angle);
					}

					// Check if the new position is within the parent circle
					if (p.dist(centerX, centerY, x, y) + dotSize / 2 <= this.radius) {
						positions.push(p.createVector(x, y));
					}
				}

				return positions;
			}

			updateGroup() {
				let value = String(this.obj[selectedVar]).replace(/[^0-9.]/g, '');

				if (value === '') {
					this.target = p.createVector(w / 2, -h);
				} else {
					const num = Number(value);
					this.varPct = num / 100;
					let y = 50;
					if (this.varPct > .5) {
						y = h - 50;
					}
					let x = p.random(0 + dotSize, w - dotSize);
					this.target = p.createVector(
						x, y
						//p.constrain(w/2, this.radius, w - this.radius),
						//h - p.constrain(this.varPct * h, this.radius, h - this.radius)
						);
				}
			}

			update() {
				let desired = p.Vector.sub(this.target, this.center);
				let distance = desired.mag();

				let speed = this.maxSpeed;
				if (distance < 100) {
					speed = p.map(distance, 0, 100, 0.5, this.maxSpeed); // Ensuring a minimum speed
				}
				desired.setMag(speed);

				let steer = p.Vector.sub(desired, this.velocity);
				steer.limit(this.maxForce);

				this.acceleration.add(steer);
				this.velocity.add(this.acceleration);
				this.velocity.limit(this.maxSpeed);

				let prevCenter = this.center.copy();

				this.center.add(this.velocity);

				this.acceleration.mult(0);

				// Only constrain the circle if the target is within the screen bounds
				if (this.target.y >= 0 && this.target.y <= h && this.target.x >= 0 && this.target.x <= w) {
					this.center.x = p.constrain(this.center.x, this.radius, w - this.radius);
					this.center.y = p.constrain(this.center.y, this.radius, h - this.radius);
				}

				// Update all persons' positions relative to the new circle center
				this.updatePeoplePositions(prevCenter, this.center);
			}

			updatePeoplePositions(prevCenter, newCenter) {
				let maxRadius = this.radius - dotSize; // Maximum radius for the people to be within the circle
				let currentRadius = dotSize * 1.2; // Start just outside the center ellipse, with minimal spacing
				let angleIncrement;
				let ellipsesInCurrentCircle;
				let angle = 0;

				let newPosition = newCenter.copy(); // Start from the center

				for (let i = 0; i < this.peoplePositions.length; i++) {
					// Set the current position
					this.peoplePositions[i] = newPosition.copy();

					// Calculate the number of ellipses in the current circle
					ellipsesInCurrentCircle = Math.floor(p.TWO_PI * currentRadius / (dotSize*1.2)); // Number of ellipses that can fit in the circumference
					angleIncrement = p.TWO_PI / ellipsesInCurrentCircle;

					// Move to the next position on the current circle
					angle += angleIncrement;

					// If a full circle is completed, move to the next concentric circle
					if (angle >= p.TWO_PI*1.0001) {
						angle = 0;
						currentRadius += dotSize * 1; // Move to the next concentric circle
						// If we exceed the max radius, stop adding more ellipses
						if (currentRadius > maxRadius) {
							break;
						}
					}

					// Calculate the new position based on the current radius and angle
					let offsetX = currentRadius * p.cos(angle);
					let offsetY = currentRadius * p.sin(angle);
					newPosition = p.createVector(newCenter.x + offsetX, newCenter.y + offsetY);
				}
			}

			collide(other) {
			    // Check if either circle's target is off the screen
				if ((this.target.y < 0 || this.target.y > h || this.target.x < 0 || this.target.x > w) ||
					(other.target.y < 0 || other.target.y > h || other.target.x < 0 || other.target.x > w)) {
			        return; // Skip collision detection if the target is off-screen
				}

				let distance = p.Vector.dist(this.center, other.center);
				let minDist = this.radius + other.radius;

				if (distance < minDist) {
				    // Calculate the overlap distance
					let overlap = minDist - distance;

				    // Calculate the direction of the overlap
					let direction = p.Vector.sub(other.center, this.center);
					direction.normalize();

				    // Apply the correction more aggressively
					let correction = direction.copy().mult(overlap / 20);
					this.center.sub(correction);
					other.center.add(correction);

				    // Reset velocities to 0 to stop movement after correction
					this.velocity.mult(0);
					other.velocity.mult(0);

				        // Adjust targets to favor larger circles towards the center
					if (this.radius >= other.radius) {
				        // Move this circle's target closer to the center
						this.target.sub(correction);
				        // Move the other circle's target outward
						other.target.add(correction);
					} else {
				        // Move this circle's target outward
						this.target.add(correction);
				        // Move the other circle's target closer to the center
						other.target.sub(correction);
					}

				        // Ensure both circles and their targets remain within the canvas bounds
					this.center.x = p.constrain(this.center.x, this.radius, w - this.radius);
					this.center.y = p.constrain(this.center.y, this.radius, h - this.radius);
					other.center.x = p.constrain(other.center.x, other.radius, w - other.radius);
					other.center.y = p.constrain(other.center.y, other.radius, h - other.radius);

					this.target.x = p.constrain(this.target.x, this.radius, w - this.radius);
					this.target.y = p.constrain(this.target.y, this.radius, h - this.radius);
					other.target.x = p.constrain(other.target.x, other.radius, w - other.radius);
					other.target.y = p.constrain(other.target.y, other.radius, h - other.radius);
				}
			}

		display() {
				// p.fill(200, 200, 255, 100);
			p.fill(0,200,200, 100);
			p.textAlign("CENTER","TOP");
			p.noStroke();
			p.ellipse(this.center.x, this.center.y, this.radius*1.1, this.radius*1.1);
				// Display each Person as a small circle

			for (let i = 0; i < this.peoplePositions.length; i++) {
				if (i/this.obj.dots < this.varPct) {
					p.fill(0,150,150);
				} else {
					p.fill(0,200,200);
				}
				p.ellipse(this.peoplePositions[i].x, this.peoplePositions[i].y, dotSize, dotSize);
			}
		}

		displayText() {
			p.fill(0);
			p.stroke(255);
			p.text(this.obj.OCCUPATION,this.center.x - 50, this.center.y, 100, 100);	
		}
	}
};
</script>

<P5 {sketch} />

<style>
	
</style>