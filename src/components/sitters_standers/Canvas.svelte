<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let bg, data, currentData, questions, copy, currentQuestionNum, currentStageNumber, minmax, minIndicies, maxIndicies, searchValue, explored, x_axis_variable, x_axis_variable_range, currentVar, reset;
	let prevStageNumber = currentStageNumber;
	let circles = [];
	let dotSize = 5; 
	let new_currentVar = -1;
	let prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
	let multiplier = 0.038;
	let guidedTour = true;
	let zoomedGuidedTour = true;
	let job_hl_index = 0;
	let userControl = false;
	let zoom = 4;       // Zoom level
	let zoomTarget = 4;
	let zoomMinMax = [0.2, 6];
	let offsetX = 0;    // X offset for panning
	let offsetXTarget = 0;
	let offsetY = 0;    // Y offset for panning
	let offsetYTarget = 0;
	let startX, startY; // To store the starting position of dragging
	let startTouches = []; // To store touch points for pinch zoom
	let previousDistance = null; // To store the previous distance between two touches
	let atlasGrotesk;
	let marginTop = 60;
	const marginBottom = 200;
	let divider = 1; // divides number of dots for smaller screens
	let moneyMinMax = [999999,-999999];
	let stage;
	let maxSpeed = 15;
	let maxForce = 5;
	const paddingX = 20;
	const no_hl_list = ["A_MEAN","INJURY_RATE"]; // things to not highlight
	let no_hl = false;
	const sketch = (p) => {

		p.preload = () => {
			atlasGrotesk = p.loadFont('assets/app/AtlasGrotesk-Regular-Web.otf'); // TiemposTextWeb-Regular.otf
		}
		p.setup = () => {
			w = p.constrain(p.windowWidth,800,6000);
			h = p.windowHeight - marginBottom;
			dotSize = 4;
			p.createCanvas(w, h);
			

			for (let i = 0; i < data.length; i++) {
				let circle = new Circle(data[i], i);
				circles.push(circle);
			}
			p.canvas.oncontextmenu = () => false;
			resize();
		};
		p.draw = () => {

			if (reset) {
				console.log("reset")
				p.reset();
				reset = false;
			}
			if (currentStageNumber != prevStageNumber || zoomedGuidedTour) {
				userControl = false;
				prevStageNumber = currentStageNumber;
				if (no_hl_list.indexOf(currentVar) != -1) {
					no_hl = true;
				} else {
					no_hl = false;
				}
			}
			if (!userControl) {
				setStage();
			}
			p.clear();
			// p.smooth();
			p.background("#150317");
			
			p.push();
			p.translate(offsetX, offsetY);
			p.scale(zoom);

			if (currentStageNumber != 0) {
				p.stroke("#5c395c");
				p.strokeWeight(0.5/zoom);
				p.line(w/2,-h*3,w/2,h*3);
				p.textSize(15/zoom);
				p.fill(255,255,255);
				p.textAlign(p.RIGHT);
				p.textFont('Arial');
				p.text("←",w/2 - (94/zoom), 23/zoom);
				p.textAlign(p.LEFT);
				p.text("→",w/2 + (106/zoom), 23/zoom);

				p.textFont(atlasGrotesk);
				p.textAlign(p.RIGHT);
				p.text("Sitters",w/2 - (34/zoom), (23/zoom));
				p.textAlign(p.LEFT);
				p.text("Standers",w/2 + (30/zoom), (23/zoom));
			}

			
			for (let i = 0; i < circles.length; i++) {
				if (checkDisplay(i) && data[i].score != -1) {
					circles[i].updateGroup();
					circles[i].update();
					circles[i].display();
					for (let j = 0; j < circles.length; j++) {
						if (j != i && data[j].score != -1 && data[i].score != -1) {
							circles[i].collide(circles[j]);
						}
					}
				}
			}

			for (let i = 0; i < circles.length; i++) {
				if (checkDisplay(i) && data[i].score != -1) {
					circles[i].hovered = false;
					// Adjust the mouse position to account for zoom and pan
					let adjustedMouseX = (p.mouseX - offsetX) / zoom;
					let adjustedMouseY = (p.mouseY - offsetY) / zoom;

				    // Check hover using adjusted mouse position
					if (p.dist(adjustedMouseX, adjustedMouseY, circles[i].center.x, circles[i].center.y) < circles[i].radius / 2) {
						circles[i].hovered = true;
					}
					circles[i].displayText(circles);
				}
			}

			if (currentVar != new_currentVar) {
				prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
				new_currentVar = currentVar;
			}
			p.pop();
		};

		function checkDisplay(n) {
			zoomedGuidedTour = true;
			guidedTour = true;
			stage = copy.story[currentStageNumber].stage;
			if (stage == "explore") {
				return true;
			}
			if (stage == "one_similar_job" && n == maxIndicies[0]) {
				return true;
			}
			if (stage == "other_similar_jobs" && maxIndicies.includes(n)) {
				return true;
			}
			if (stage == "other_similar_jobs_you" && (maxIndicies.includes(n) || n == data.length-1)) {
				return true;
			}
			if (stage == "other_dissimilar_jobs" && (minIndicies.includes(n) || maxIndicies.includes(n))) {
				return true;
			}

			if (stage == "other_dissimilar_jobs_you" && (minIndicies.includes(n) || maxIndicies.includes(n) || n == data.length-1)) {
				return true;
			}

			if (stage == "all_jobs_zoomout") {
				zoomedGuidedTour = false;
				return true;
			}
			if (stage == "all_jobs_hl") {
				zoomedGuidedTour = false;
				return true;
			}
			if (stage == "all_jobs") {
				zoomedGuidedTour = false;
				return true;
			}
			if (stage == "explore") {
				guidedTour = false;
				return true;
			}
			
			return false;
		}

		function setStage() {
			if (copy.story[currentStageNumber].stage == "explore") {
				return;
			}
			if (copy.story[currentStageNumber].stage == "one_similar_job") {
				centerAndZoomOnCoordinate(circles[maxIndicies[0]].center.x, circles[maxIndicies[0]].center.y, 4);
			}
			if (copy.story[currentStageNumber].stage == "other_similar_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, 0.9);
			}
			if (copy.story[currentStageNumber].stage == "other_dissimilar_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, 0.9);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_zoomout") {
				centerAndZoomOnCoordinate(w/2, h/2, 0.95);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_hl") {
				centerAndZoomOnCoordinate(circles[job_hl_index].center.x, circles[job_hl_index].center.y, 0.95);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs") {
				let nudge = 0;
				if (w > 840) {
					nudge = w/50;
				}
				centerAndZoomOnCoordinate(w/2 + nudge, h/2, 0.95);
			}
		}

		function centerAndZoomOnCoordinate(targetX, targetY, targetZoom) {
		    // Set the zoom level to the target zoom
			zoomTarget = targetZoom;

		    // // Calculate the offset to center the target coordinate on the canvas
			offsetXTarget = p.width / 2 - targetX * zoomTarget;
			offsetYTarget = p.height / 2 - targetY * zoomTarget;
			let lerpSpeed = 0.05;
			maxSpeed = 12;
			maxForce = 5;
			if (currentStageNumber == 0) {
				lerpSpeed = 0.1;
				maxSpeed = 15;
				maxForce = 10;
			} else if (zoomedGuidedTour || userControl) {
				lerpSpeed = 0.05;
				maxSpeed = 2;
				maxForce = 1;
			} 
			zoom = p.lerp(zoom, zoomTarget, lerpSpeed);
			offsetX = p.lerp(offsetX, offsetXTarget, lerpSpeed);
			offsetY = p.lerp(offsetY, offsetYTarget, lerpSpeed);
		}

		// Function to check if mouse is over the p5.js canvas
		function isMouseOverCanvas() {
		    // Get the element under the mouse
			let element = document.elementFromPoint(p.mouseX + p.canvas.offsetLeft, p.mouseY + p.canvas.offsetTop);
		    // Return true if the element is the p5.js canvas
			return element === p.canvas;
		}

		// Function to constrain offset values within the visible canvas bounds
		function constrainOffsets() {
			const maxOffsetX = (w * zoom) / 1.5;
			const maxOffsetY = (h * zoom) / 1.5;

		    // Clamp offsetX and offsetY to prevent moving outside the canvas
			offsetX = p.constrain(offsetX, -maxOffsetX, maxOffsetX);
			offsetY = p.constrain(offsetY, -maxOffsetY, maxOffsetY);
		}

		// Handle zooming with mouse wheel
		p.mouseWheel = (event) => {
		    if (!isMouseOverCanvas()) return; // Prevent zooming if not over the canvas
		    userControl = true;
		    explored = true;
		    
		    let scrollVelocity = Math.abs(event.delta); // Use absolute value of delta
		    let baseZoomSpeed = 0.05 * zoom; // Base zoom speed proportional to current zoom
		    let zoomSpeed = baseZoomSpeed * (scrollVelocity / 100); // Faster scroll leads to faster zoom

		    let previousZoom = zoom;

		    if (event.delta > 0) {
		        zoom = p.constrain(zoom - zoomSpeed, zoomMinMax[0], zoomMinMax[1]); // Zoom out
		    } else {
		        zoom = p.constrain(zoom + zoomSpeed, zoomMinMax[0], zoomMinMax[1]); // Zoom in
		    }

		    // Calculate the difference in zoom and adjust the offset
		    let zoomChange = zoom / previousZoom;

		    // Focus zoom on mouse position
		    offsetX = p.mouseX - (p.mouseX - offsetX) * zoomChange;
		    offsetY = p.mouseY - (p.mouseY - offsetY) * zoomChange;

		    // Constrain offsets so you can't pan outside the canvas
		    constrainOffsets();
		};

		// Handle panning with mouse drag
		p.mousePressed = () => {
		    if (!isMouseOverCanvas()) return; // Prevent panning if not over the canvas
		    startX = p.mouseX - offsetX;
		    startY = p.mouseY - offsetY;
		};

		// Handle panning with mouse drag
		p.mouseDragged = () => {
		    if (!isMouseOverCanvas()) return; // Prevent dragging if not over the canvas
		    userControl = true;
		    explored = true;
		    
		    offsetX = p.mouseX - startX;
		    offsetY = p.mouseY - startY;

		    // Constrain offsets so you can't pan outside the canvas
		    constrainOffsets();
		};

		let pinchZooming = false;
		let wasPinching = false;

		p.touchStarted = () => {
			userControl = true;
			explored = true;
		    if (!isMouseOverCanvas()) return; // Prevent touch actions if not over the canvas
		    if (p.touches.length === 1 && !pinchZooming) {
		        // Single touch for panning, allowed if not in pinch zoom mode
		    	startX = p.touches[0].x - offsetX;
		    	startY = p.touches[0].y - offsetY;
		    } else if (p.touches.length === 2) {
		        // Store the initial positions of two touches for pinch zoom
		    	pinchZooming = true;
		    	wasPinching = true;
		    	previousDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);
		    }
		};

		// Handle touch-based pinch zoom and pan
		p.touchMoved = () => {
		    if (!isMouseOverCanvas()) return false; // Prevent touch movement if not over the canvas
		    userControl = true;
		    explored = true;
		    
		    if (p.touches.length === 1) {
		    	if (pinchZooming) {
		    		startX = p.touches[0].x - offsetX;
		    		startY = p.touches[0].y - offsetY;
		            pinchZooming = false; // Exit pinch zoom mode
		        }
		        // Pan with single finger swipe
		        offsetX = p.touches[0].x - startX;
		        offsetY = p.touches[0].y - startY;
		    } else if (p.touches.length === 2) {
		    	let currentDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);

		    	if (previousDistance) {
		    		let zoomChange = currentDistance / previousDistance;
		    		zoom = p.constrain(zoom * zoomChange, zoomMinMax[0], zoomMinMax[1]);

		    		let midX = (p.touches[0].x + p.touches[1].x) / 2;
		    		let midY = (p.touches[0].y + p.touches[1].y) / 2;

		    		offsetX = (midX - offsetX) * (1 - zoomChange) + offsetX;
		    		offsetY = (midY - offsetY) * (1 - zoomChange) + offsetY;
		    	}

		    	previousDistance = currentDistance;
		    	pinchZooming = true;
		    }

		    // Constrain offsets so you can't pan outside the canvas
		    constrainOffsets();

		    return false; // Prevent default behavior (like scrolling the page)
		};

		p.touchEnded = () => {
			userControl = true;
			explored = true;

			if (p.touches.length === 1 && wasPinching) {
		        // If transitioning from a pinch, avoid jumping offsets
				startX = p.touches[0].x - offsetX;
				startY = p.touches[0].y - offsetY;
		        wasPinching = false; // Reset pinch flag
		    }

		    // Reset after touch ends
		    previousDistance = null;
		};

		let resizeTimeout;

		p.windowResized = () => {
			resize();
		};

		function resize() {
			if (p.windowWidth < 860) {
				w = p.windowWidth;
				h = p.windowHeight - marginBottom + 80;
				marginTop = 70;
			} else {
				w = p.windowWidth - 250;
				h = p.windowHeight;
				marginTop = 60;
			}
			
			dotSize = 4;
			if (p.windowWidth < 1500) {
				dotSize = 3;
			}
			if (p.windowWidth < 800) {
				divider = 2;
			}
			p.resizeCanvas(w, h);

			clearTimeout(resizeTimeout);

			resizeTimeout = setTimeout(() => {
				for (let i = 0; i < circles.length; i++) {
					circles[i].updateGroup();
				}
			}, 100);
		}

		class Circle {
			constructor(obj, index) {
				this.obj = obj;
				this.index = index;
				this.radius = this.calculateOptimalRadius(obj.dots / divider, dotSize);
				if (obj.OCCUPATION == "You") {
					this.radius = 10;
				}
				this.currentColors = Array(this.obj.dots).fill(p.color("#523c50"));
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					-200
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.peoplePositions = this.calculatePeoplePositions(obj.dots / divider);
				this.hovered = false;
				this.maxSpeed = 15;
				this.maxForce = 5;
				this.score = 0;
				this.textDisplayed = false;
				this.varPct = 0; // Initialize varPct
				this.targetVarPct = 0;
        		this.prevVarPct = 0; // Initialize previous varPct
        		this.xValue = obj[x_axis_variable];
        	}

        	calculateOptimalRadius(totalDots, dotSize) {
        		if (totalDots <= 1) {
			        return 1 * dotSize*1.9; // Only the center dot, hence 1 circle
			    }

			    // The number of concentric circles is the largest floor value of sqrt(i) for i up to totalPeople - 1
			    let lastIndex = totalDots - 1;
			    let numberOfCircles = Math.floor(Math.sqrt(lastIndex)) + 2; // Add 1 because the first circle is counted from 0
			    return numberOfCircles*dotSize*1.3;
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

					positions.push(p.createVector(x, y));
				}
				return positions;
			}

			updateGroup() {
				this.score = 0;
				for (let i = 0; i < currentQuestionNum; i++) {
					const question = questions[i];
					let value = String(this.obj[question]).replace(/[^0-9.]/g, '');
					if (value !== "" && Number(value) > 0) {
						if (
							Number(value) > Number(copy.questions[i].threshold) &&
							currentData[i] === 0 &&
							value > 0
							) {
							this.score = 1;
					}
				}
			}

		        // Smoothly transition varPct using lerp for a smoother visual effect
			this.targetVarPct = Number(String(this.obj[currentVar]).replace(/[^0-9.]/g, ''));
		        this.varPct = p.lerp(this.varPct, this.targetVarPct, 0.1); // Gradually move towards the target value
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
				this.maxSpeed = maxSpeed;
				this.maxForce = maxForce;
				this.radius = this.calculateOptimalRadius(this.obj.dots / divider, dotSize);
				this.peoplePositions = this.calculatePeoplePositions(this.obj.dots / divider);
				if (this.obj.OCCUPATION == "You") {
					this.radius = 10;
				}
			    // Set the target x and y positions based on data
				if (w > 860) {
					this.target.y = p.constrain(
						p.map(data[this.index].score, minmax[0], minmax[1], h-30, marginTop),
						marginTop,
						h
						); 
				} else {
					this.target.y = p.constrain(
						p.map(data[this.index].score, minmax[0], minmax[1], h-30, marginTop),
						marginTop,
						h-150
						); 
				}

				this.target.x = p.constrain(
					p.map(data[this.index][x_axis_variable], x_axis_variable_range[x_axis_variable][0],  x_axis_variable_range[x_axis_variable][1], paddingX, w-paddingX),
					paddingX,
					w-paddingX
					);

			    // Calculate the direction to the target position
				let directionToTarget = p.createVector(
					this.target.x - this.center.x,
					this.target.y - this.center.y
					);
			    directionToTarget.mult(0.05); // Control the speed of movement toward the target

			    let friction = this.velocity.copy();
			    friction.mult(-0.3); // Apply more friction to further slow down movement

			    // Apply movement toward the target and friction
			    this.acceleration.add(directionToTarget);
			    this.acceleration.add(friction);

			    // Limit the acceleration to this.maxForce
			    if (this.acceleration.mag() > this.maxForce && !zoomedGuidedTour) {
			    	this.acceleration.setMag(this.maxForce);
			    }

			    // Update velocity with acceleration
			    this.velocity.add(this.acceleration);

			    // Limit the velocity to this.maxSpeed
			    if (this.velocity.mag() > this.maxSpeed && !zoomedGuidedTour) {
			    	this.velocity.setMag(this.maxSpeed);
			    }

			    // Update center position with the new velocity
			    this.center.add(this.velocity);

			    // Constrain the circle within screen bounds
			    this.center.x = p.constrain(this.center.x, this.radius, w - this.radius);
			    this.center.y = p.constrain(this.center.y, this.radius + marginTop, h - this.radius);

			    // Apply stronger damping to simulate energy loss
			    this.velocity.mult(0.9); // Stronger damping to slow down circles faster

			    // Reset acceleration for the next frame
			    this.acceleration.mult(0);

			    // Update positions of people within the circle
			    this.updatePeoplePositions(this.center);
			}

			collide(other) {
				let distance = p.Vector.dist(this.center, other.center);
			    let minDist = (this.radius/1.4 + other.radius/1.4); // The minimum distance to prevent overlap
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
			        let bounceFactor = 0.3; // Reduce bounce even further
			        let bounceEffect = direction.copy().mult(bounce * bounceFactor);
			        this.velocity.sub(bounceEffect);
			        other.velocity.add(bounceEffect);

			        // Apply stronger damping to reduce velocity after bounce
			        let dampingFactor = 0.15; // Stronger damping for faster energy dissipation
			        this.velocity.mult(dampingFactor);
			        other.velocity.mult(dampingFactor);

			        // Gradually reduce velocity to encourage settling
			        this.velocity.mult(0.92);
			        other.velocity.mult(0.92);

			        // Stop the circles if their velocity is below a slightly higher threshold
			        let velocityThreshold = 0.3; // Increased threshold for stopping
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
				p.ellipseMode(p.CENTER);
				p.strokeWeight(0.4 / zoom);

				p.stroke("#947594");

				if (this.hovered || this.textDisplayed) {
					p.stroke("#ffffff");
					p.strokeWeight(0.4 / zoom);
				}

				if (searchValue == this.obj.OCCUPATION) {
					job_hl_index = this.index;
					p.stroke("#ffffff");
					p.strokeWeight(1 / zoom);
				}

				if (this.obj.OCCUPATION == "You") {
					p.stroke(252, 186, 3);
					p.fill(252, 186, 3);
					p.strokeWeight(1 / zoom);
					p.circle(this.center.x, this.center.y, 10);
				} else if (data[this.index].score != -1) {
					if (searchValue == this.obj.OCCUPATION) {
						p.stroke(252, 186, 3);
						p.strokeWeight(2 / zoom);
					}
					p.circle(this.center.x, this.center.y, this.radius);	
			        const transitionSpeed = 0.3; // Smooth transition speed
			        const filledColor = p.color("#ff69f2");
			        const unfilledColor = p.color("#523c50");

			        // Calculate how many dots to fill based on smoothed varPct
			        let dotsToFill = Math.round(this.obj.dots / divider * (this.varPct / 100));
			        if (no_hl) {
			        	dotsToFill = 0;
			        }
			        // Display each dot with smooth color transitions
			        for (let i = 0; i < this.peoplePositions.length; i++) {
			        	p.noStroke();

			            // Determine target color based on the smoothed fill state
			        	let targetColor = i < dotsToFill ? filledColor : unfilledColor;

			            // Smoothly interpolate the current color towards the target color
			        	this.currentColors[i] = p.lerpColor(this.currentColors[i], targetColor, transitionSpeed);

			            // Set the fill to the interpolated color
			        	p.fill(this.currentColors[i]);
			        	p.circle(this.peoplePositions[i].x, this.peoplePositions[i].y, dotSize);
			        }
			    }
			}

			checkTextOverlap(otherCircles) {
			    // Approximate text height and width based on zoom and font size
				let scaledFontSize = h / 12 / zoom * 2;
				if (guidedTour && zoomedGuidedTour) {
					scaledFontSize = h / 10 / zoom;
				}

			    const avgCharWidth = 7; // Assume an average character width for optimization
			    const textWidth = avgCharWidth * this.obj.OCC_SHORT.length / zoom; // Approximate text width
			    const textHeight = scaledFontSize;

			    // Position the text centered above the circle
			    const leftX = this.center.x - textWidth / 2;
			    const rightX = this.center.x + textWidth / 2;
			    const topY = this.center.y - this.radius / 2 - (3 / zoom) - textHeight;
			    const bottomY = topY + textHeight;

			    // Check for overlap with other circles' text using simple bounding box collision
			    for (let other of otherCircles) {
			        if (other === this || !other.textDisplayed) continue; // Skip itself or non-displayed texts

			        const otherTextWidth = avgCharWidth * other.obj.OCC_SHORT.length / zoom;
			        const otherLeftX = other.center.x - otherTextWidth / 2;
			        const otherRightX = other.center.x + otherTextWidth / 2;
			        const otherTopY = other.center.y - other.radius / 2 - (3 / zoom) - textHeight;
			        const otherBottomY = otherTopY + textHeight;

			        // Approximate overlap check using bounding boxes
			        if (leftX < otherRightX && rightX > otherLeftX && topY < otherBottomY && bottomY > otherTopY) {
			            return true; // Early exit if overlap is detected
			        }
			    }
			    
			    return false; // No overlap
			}


			displayText(otherCircles) {
			    const maxAlpha = 255; // Full opacity
			    const fadeSpeed = 15; // Speed of fading in and out

			    // Initialize alpha if not already defined
			    if (this.alpha === undefined) {
			        this.alpha = 0; // Start with fully transparent text
			    }

			    // Determine if prioritization should occur based on currentVar and varPct
			    let prioritizeVarPct = false;
			    const varPctThreshold = 20; // Threshold for varPct when currentVar is not empty

			    // Check if currentVar is empty or null, or if the circle is hovered
			    if (this.obj[currentVar] > 30 && ((this.xValue > 50 && bg == "stand") || (this.xValue < 50 && bg == "sit")) ) {
			        prioritizeVarPct = true; // Only prioritize and show when varPct > 20
			    } else if (!currentVar || this.hovered) {
			        prioritizeVarPct = true; // If currentVar is empty or the circle is hovered
			    }

			    // Determine shouldDisplayText based on prioritization rules, with hovered overriding
			    let shouldDisplayText = 
			    (this.hovered ||  // Hovering should override everything
			    	(prioritizeVarPct && 
			    		((this.radius > 30 && !this.checkTextOverlap(otherCircles)) || 
			    			(!this.checkTextOverlap(otherCircles) && (guidedTour || zoomedGuidedTour)))));

			    // Search value checks
			    if (searchValue != "" && searchValue != this.obj.OCCUPATION) {
			    	shouldDisplayText = false;
			    } else if (searchValue != "" && searchValue == this.obj.OCCUPATION) {
			    	shouldDisplayText = true;
			    }

			    const textWidth = p.textWidth(this.obj.OCC_SHORT);

			    // Prevent text display if it extends outside canvas bounds when zoom is near 1
			    if ((this.center.x - textWidth / 2 < 0 || this.center.x + textWidth / 2 > w) && Math.abs(zoom - 1) < 0.1) {
			    	shouldDisplayText = false;
			    }

			    // Always highlight "You"
			    if (this.obj.OCCUPATION == "You" || this.hovered || searchValue == this.obj.OCCUPATION) {
			    	shouldDisplayText = true;
			    }

			    // Gradually fade in or out the text
			    if (shouldDisplayText) {
			        this.alpha = Math.min(this.alpha + fadeSpeed, maxAlpha); // Fade in
			    } else {
			        this.alpha = Math.max(this.alpha - fadeSpeed, 0); // Fade out
			    }

			    // Only draw text if it's at least partially visible
			    if (this.alpha > 0) {
			        // Set the fill color with current alpha for fading effect
			    	if (this.obj.OCCUPATION == "You" || searchValue == this.obj.OCCUPATION) {
			    		p.fill(252, 186, 3, this.alpha);
			    	} else {
			    		// p.fill(161, 142, 171, this.alpha);    
			    		p.fill(191, 172, 201, this.alpha);    
			    	}

			        const maxTextWidth = 100 / zoom; // Scale the maximum width based on zoom
			        const scaledFontSize = 12 / zoom; // Adjust font size based on zoom

			        // Set the font size before displaying text
			        p.textSize(scaledFontSize);

			        // Measure the width of the text within the max width
			        const textWidth = p.textWidth(this.obj.OCC_SHORT); // Actual width of the text

			        // Approximate text height based on font size
			        const lineHeight = scaledFontSize; // Set line height equal to font size
			        const textHeight = lineHeight; // Single line height for this text

			        // Align the text to be centered and bottom aligned
			        p.textAlign(p.CENTER, p.BOTTOM);

			        // Position the text exactly centered above the circle
			        let xPos = this.center.x; // Center the text horizontally with the circle
			        let yPos = this.center.y - this.radius / 2 - (3 / zoom); // Place the text 3 pixels above the circle, scaled by zoom
			        p.stroke("#150317");
			        p.strokeWeight(4/zoom);
			        if (searchValue == "") {
			        	p.text(this.obj.OCC_SHORT, xPos, yPos);
			        } else if (searchValue == this.obj.OCCUPATION && data[this.index].score != -1) {
			        	p.text(this.obj.OCC_SHORT, xPos, yPos);
			        }


			        this.textDisplayed = true; // Mark text as displayed
			    } else {
			        this.textDisplayed = false; // Mark text as hidden
			    }
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