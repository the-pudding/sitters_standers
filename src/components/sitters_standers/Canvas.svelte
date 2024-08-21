<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let data;

	let allPeople = [];
	const dotPerPerson = 50000;
	let dotSize = 10;

	const sketch = (p) => {
		p.setup = () => {
			w = p.windowWidth;
			h = p.windowHeight - 6;
			p.createCanvas(w, h);
			let counter = 0;
			for (let i = 0; i < data.length; i++) {
				for (let j = 0; j < data[i].dots; j++) {
					allPeople[counter] = new Person(data[i], counter);
					counter += 1;
				}
			}
		};

		p.draw = () => {
			p.clear();
			p.background(240, 240, 240);
			p.fill(255);
			p.noStroke();
    		// Check for collisions between every pair
			for (let i = 0; i < allPeople.length; i++) {
				for (let j = i + 1; j < allPeople.length; j++) {
					if (Math.abs(allPeople[i].location.x - allPeople[j].location.x) < 15 && Math.abs(allPeople[i].location.y -allPeople[j].location.y) < 15) {
						allPeople[i].collide(allPeople[j]);
					}
				}
				allPeople[i].update();
				allPeople[i].display();
			}
		};

		let resizeTimeout;

		p.windowResized = () => {
		    w = p.windowWidth;
		    h = p.windowHeight - 6;
		    p.resizeCanvas(w, h);

		    // Clear the previous timeout if the windowResized function is triggered again
		    clearTimeout(resizeTimeout);

		    // Set a new timeout to wait 0.3 seconds before running the resize loop
		    resizeTimeout = setTimeout(() => {
		        for (let i = 0; i < allPeople.length; i++) {
		            allPeople[i].resize();
		        }
		    }, 300); // 300 milliseconds = 0.3 seconds
		};

		class Person {
		    constructor(obj, personNumber) {
		        this.n = personNumber;
		        this.obj = obj;
		        this.location = p.createVector(w * Math.random(), h * Math.random());
		        this.velocity = p.createVector(0, 0);
		        this.acceleration = p.createVector(0, 0);
		        this.target = p.createVector(dotSize + p.random(w - dotSize*2), dotSize + p.random(h - dotSize*4));
		        this.maxSpeed = 8;
		        this.maxForce = 0.4;
		    }

		    update() {
		        // Calculate the desired vector pointing from location to target
		        let desired = p.Vector.sub(this.target, this.location);
		        let distance = desired.mag();

		        // If we are close to the target, slow down
		        let speed = this.maxSpeed;
		        if (distance < 100) {
		            speed = p.map(distance, 0, 100, 0, this.maxSpeed);
		        }
		        desired.setMag(speed);

		        // Steering = Desired - Velocity
		        let steer = p.Vector.sub(desired, this.velocity);
		        steer.limit(this.maxForce);

		        // Apply acceleration
		        this.acceleration.add(steer);
		        // let nudge = p.createVector(p.random(-0.2, 0.2), p.random(-0.2, 0.2));
		        // this.acceleration.add(nudge);

		        // Update velocity
		        this.velocity.add(this.acceleration);

		        // Limit velocity to max speed
		        this.velocity.limit(this.maxSpeed);

		        // Update location
		        this.location.add(this.velocity);

		        // Reset acceleration to 0 each cycle
		        this.acceleration.mult(0);

		    }

		    display() {
		        // p.rect(this.location.x+2, this.location.y+5, dotSize/4, dotSize/4);
		        // p.rect(this.location.x-4, this.location.y+5, dotSize/4, dotSize/4);
		        p.fill(0);
		        p.rect(this.location.x, this.location.y, dotSize/2, dotSize);
		        p.fill(100,100,0);
		        p.rect(this.location.x, this.location.y, dotSize/2, dotSize / 4);
		    }

		    resize() {
		    	this.target = p.createVector(dotSize + p.random(w - dotSize), dotSize + p.random(h - dotSize));
		    }

		    collide(other) {
			    let distance = p.Vector.dist(this.location, other.location);
			    let minDist = dotSize;

			    if (distance < minDist) {
			        // Calculate the normal vector
			        let normal = p.Vector.sub(other.location, this.location);
			        normal.normalize();

			        // Calculate relative velocity
			        let relativeVelocity = p.Vector.sub(this.velocity, other.velocity);
			        let speed = relativeVelocity.dot(normal);

			        if (speed > 0) {
			            return; // They are moving apart
			        }

			        // Calculate the impulse scalar
			        let impulse = (2 * speed) / (1 + 1); // assuming equal mass

			        // Apply the impulse to both objects
			        this.velocity.add(p.Vector.mult(normal, -impulse));
			        other.velocity.add(p.Vector.mult(normal, impulse));

			        // Resolve overlap (push them apart)
			        let overlap = minDist - distance;
			        let correction = normal.copy().mult(overlap / 2); // divide correction equally
			        this.location.sub(correction);
			        other.location.add(correction);

			        // Add a small random displacement to prevent sticking
			        let nudge = p.createVector(p.random(-1, 1), p.random(-1, 1));
			        this.location.add(nudge);
			        other.location.add(nudge.mult(-1)); // Opposite direction for the other ellipse

			        if (p.Vector.dist(this.location, this.target) < 3 || p.Vector.dist(other.location, other.target) < 3) {
					    // Generate a random displacement vector with a magnitude between 5 and 20 pixels
					    let minDisplacement = 3;
					    let maxDisplacement = 5;

					    let thisDisplacement = p.createVector(
					        p.random(minDisplacement, maxDisplacement) * (p.random() < 0.5 ? -1 : 1),
					        p.random(minDisplacement, maxDisplacement) * (p.random() < 0.5 ? -1 : 1)
					    );

					    let otherDisplacement = p.createVector(
					        p.random(minDisplacement, maxDisplacement) * (p.random() < 0.5 ? -1 : 1),
					        p.random(minDisplacement, maxDisplacement) * (p.random() < 0.5 ? -1 : 1)
					    );

					    // Move the targets by the displacement vectors, keeping them within the canvas bounds
					    this.target.add(thisDisplacement);
					    other.target.add(otherDisplacement);

					    // Ensure the new target is within canvas boundaries
					    this.target.x = p.constrain(this.target.x, dotSize, w - dotSize);
					    this.target.y = p.constrain(this.target.y, dotSize, h - dotSize);
					    
					    other.target.x = p.constrain(other.target.x, dotSize, w - dotSize);
					    other.target.y = p.constrain(other.target.y, dotSize, h - dotSize);
					}

			    }
			}
		}
	};

</script>

<P5 {sketch} />

<style>
	
</style>