<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let data, currentData, questions, copy, currentQuestionNum;

	let circles = [];
	const dotSize = 5; 
	export let currentVar;
	let new_currentVar = -1;
	let prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
	let multiplier = 0.038;

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
			p.smooth();
			p.background(255);

			for (let i = 0; i < circles.length; i++) {
				circles[i].updateGroup();
				circles[i].update();
				circles[i].display();
				for (let j = 0; j < circles.length; j++) {
					if (j != i) {
						circles[i].collide(circles[j]);
					}
				}
				
			}

			for (let i = 0; i < circles.length; i++) {
				circles[i].hovered = false;
				if (p.dist(p.mouseX, p.mouseY, circles[i].center.x, circles[i].center.y) < circles[i].radius/2) {
					circles[i].displayText();	
					circles[i].hovered = true;
				}
			}

			if (currentVar != new_currentVar) {
				// for (let i = 0; i < circles.length; i++) {
				// 	circles[i].updateGroup();
				// }
				prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
				new_currentVar = currentVar;
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
			}, 100);
		};

		class Circle {
			constructor(obj, index) {
				this.obj = obj;
				this.index = index;
				
				if (obj.dots <= 1) {
					this.radius = dotSize*3;
				} else if (obj.dots <= 7) {
					this.radius = dotSize*5;
				} else {
					this.radius = Math.sqrt(obj.TOT_EMP) * multiplier;
				}
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					p.random(this.radius, h - this.radius)
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.maxSpeed = 4;
				this.maxForce = 1;
				this.peoplePositions = this.calculatePeoplePositions(obj.dots);
				this.hovered = false;
				this.score = 0;
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
				// let value = String(this.obj[currentVar]).replace(/[^0-9.]/g, '');
				// if (value === '') {
				// 	this.target = p.createVector(w / 2, -h);
				// } else {
				// 	const num = Number(value);
				// 	this.varPct = num / 100;
				// 	let y = 0;
				// 	if (this.varPct > .5) { y = h; }
				// 	let x = this.target.x;
				// 	this.target = p.createVector(x, y);
				// }
				this.score = 0;
				for (let i = 0; i < currentQuestionNum; i++) {
					const question = questions[i];
					let value = String(this.obj[question]).replace(/[^0-9.]/g, '');
						if (value != "" && Number(value) > 0) {
							if (Number(value) > Number(copy.questions[i].threshold) && currentData[i] == 0 && value > 0) {
							this.score = 1;	
						}
					}
				}

				this.varPct = Number(String(this.obj[currentVar]).replace(/[^0-9.]/g, ''));
				this.prevVarPct = Number(String(this.obj[prevVar]).replace(/[^0-9.]/g, ''));
			}

			

			updatePeoplePositions(newCenter) {
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

			update() {
			    // Determine gravity direction based on this.varPct
				let gravityAmount = p.map(this.center.y, 100, h, 15, 0.5); 
				if (this.score == 0) {
					gravityAmount = p.map(this.center.y, h, 100, -15, -0.5); 
				}
			    let gravity = p.createVector(0, gravityAmount); // Upward if varPct < 0.5, otherwise downward
			    let friction = this.velocity.copy();
			    friction.mult(-2); // Apply a small amount of friction

			    // Apply gravity and friction
			    this.acceleration.add(gravity);
			    this.acceleration.add(friction);

			    // Update velocity with acceleration
			    this.velocity.add(this.acceleration);

			    // Update center position with the new velocity
			    this.center.add(this.velocity);

			    // Constrain the circle within screen bounds
			    this.center.x = p.constrain(this.center.x, this.radius, w - this.radius);
			    this.center.y = p.constrain(this.center.y, this.radius, h - this.radius);

			    // Apply damping to simulate energy loss
			    this.velocity.mult(0.95); // Reduce velocity slightly each frame

			    // Reset acceleration for the next frame
			    this.acceleration.mult(0);

			    // Update positions of people within the circle
			    this.updatePeoplePositions(this.center);
			}

			collide(other) {
				let distance = p.Vector.dist(this.center, other.center);
			    let minDist = (this.radius/2 + other.radius/1.5); // The minimum distance to prevent overlap

			    if (distance < minDist) {
			        // Calculate the overlap distance
			    	let overlap = minDist - distance;

			        // Calculate the direction of the overlap
			    	let direction = p.Vector.sub(other.center, this.center);
			    	direction.normalize();

			        // Apply the correction to fully separate the circles
			    	let correction = direction.copy().mult(overlap / 6);
			    	this.center.sub(correction);
			    	other.center.add(correction);

			        // Calculate relative velocity in the direction of the collision
			    	let relativeVelocity = p.Vector.sub(this.velocity, other.velocity);
			    	let bounce = relativeVelocity.dot(direction);

			        // Reduce bounce effect significantly
			        let bounceFactor = 0.2; // Further reduce bounce intensity
			        let bounceEffect = direction.copy().mult(bounce * bounceFactor);
			        this.velocity.sub(bounceEffect);
			        other.velocity.add(bounceEffect);

			        // Apply strong damping to reduce velocity after bounce
			        let dampingFactor = 0.5; // Increase damping to reduce energy retention further
			        this.velocity.mult(dampingFactor);
			        other.velocity.mult(dampingFactor);

			        // Gradually reduce velocity to encourage settling
			        this.velocity.mult(0.95);
			        other.velocity.mult(0.95);

			        // Stop the circles if their velocity is below a threshold
			        let velocityThreshold = 0.1; // Define a low velocity threshold
			        if (this.velocity.mag() < velocityThreshold) {
			        	this.velocity.set(0, 0);
			        }
			        if (other.velocity.mag() < velocityThreshold) {
			        	other.velocity.set(0, 0);
			        }
			    }
			}

			display() {
				p.noFill();
				// p.fill(0,200,200, 100);
				p.textAlign("CENTER","TOP");
				p.ellipseMode("CENTER");
				p.strokeWeight(0.4);
				p.stroke(200,200,200);
				if (this.hovered) {
					p.stroke(0)
					p.strokeWeight(2);
				}
				p.circle(this.center.x, this.center.y, this.radius);
				// Display each Person as a small circle

				const dotsToFill = Math.round(this.obj.dots * (this.varPct / 100));
				for (let i = 0; i < this.peoplePositions.length; i++) {
					p.noStroke();
					if (i < dotsToFill) {
						p.fill(0); // Fill color for dots that should be filled
					} else {
						p.fill(230, 230, 230); // Fill color for remaining dots
					}
					p.ellipse(this.peoplePositions[i].x, this.peoplePositions[i].y, dotSize, dotSize);
				}
				// p.fill(0)
				// p.text(this.varPct, this.center.x, this.center.y);
			}

			displayText() {
				p.fill(0);
				p.stroke(255);
				p.text(this.obj.OCCUPATION + "\n\n" + currentVar + "\n\n" + Number(this.varPct),20, h/2, 200, 100);
			}
		}
	};

	$: {
		currentVar, prevVar;
	}
</script>

<P5 {sketch} />

<style>
	
</style>