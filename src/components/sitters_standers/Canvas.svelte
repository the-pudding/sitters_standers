<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let data, currentData, questions, copy, currentQuestionNum, currentStageNumber, minmax, minIndicies, maxIndicies, searchValue;

	let circles = [];
	let dotSize = 5; 
	export let currentVar;
	let new_currentVar = -1;
	let prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
	let multiplier = 0.038;
	let guidedTour = true;
	let zoomedGuidedTour = true;
	let job_hl_index = 0;

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
	let marginTop = 100;
	const marginBottom = 170;



	const sketch = (p) => {

		p.preload = () => {
			atlasGrotesk = p.loadFont('assets/app/AtlasGrotesk-Regular-Web.otf');
		}
		p.setup = () => {
			w = p.constrain(p.windowWidth,800,6000);
			// h = p.windowHeight - 6;
			h = p.windowHeight - marginBottom;
			dotSize = 4;
			if (p.windowWidth < 1500) {
				dotSize = 3;
			}
			if (p.windowWidth < 600) {
				dotSize = 2;
			}

			p.createCanvas(w, h);
			

			for (let i = 0; i < data.length; i++) {
				let circle = new Circle(data[i], i);
				circles.push(circle);
			}
			p.canvas.oncontextmenu = () => false;
			resize();
		};

		p.draw = () => {
			setStage();
			p.clear();
			p.smooth();
			p.background("#150317");
			p.textFont(atlasGrotesk);
			p.push();
			p.translate(offsetX, offsetY);
			p.scale(zoom);

			for (let i = 0; i < circles.length; i++) {
				if (checkDisplay(i)) {
					circles[i].updateGroup();
					circles[i].update();
					circles[i].display();
					for (let j = 0; j < circles.length; j++) {
						if (j != i) {
							circles[i].collide(circles[j]);
						}
					}
				}
			}

			for (let i = 0; i < circles.length; i++) {
				if (checkDisplay(i)) {
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
			if (copy.story[currentStageNumber].stage == "explore") {
				return true;
			}
			if (copy.story[currentStageNumber].stage == "one_similar_job" && n == maxIndicies[0]) {
				return true;
			}
			if (copy.story[currentStageNumber].stage == "other_similar_jobs" && (minIndicies.includes(n) || maxIndicies.includes(n))) {
				return true;
			}
			if (copy.story[currentStageNumber].stage == "other_dissimilar_jobs" && (minIndicies.includes(n) || maxIndicies.includes(n))) {
				return true;
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_zoomout") {
				zoomedGuidedTour = false;
				return true;
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_hl") {
				zoomedGuidedTour = false;
				return true;
			}
			if (copy.story[currentStageNumber].stage == "all_jobs") {
				zoomedGuidedTour = false;
				return true;
			}
			if (copy.story[currentStageNumber].stage == "explore") {
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
				centerAndZoomOnCoordinate(circles[maxIndicies[0]].center.x, circles[maxIndicies[0]].center.y, 0.9);
			}
			if (copy.story[currentStageNumber].stage == "other_dissimilar_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, .7);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_zoomout") {
				centerAndZoomOnCoordinate(w/2, h/2, 0.6);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs_hl") {
				centerAndZoomOnCoordinate(circles[job_hl_index].center.x, circles[job_hl_index].center.y, 1.2);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, 1);
			}
		}

		function centerAndZoomOnCoordinate(targetX, targetY, targetZoom) {
		    // Set the zoom level to the target zoom
			zoomTarget = targetZoom;

		    // // Calculate the offset to center the target coordinate on the canvas
			offsetXTarget = p.width / 2 - targetX * zoomTarget;
			offsetYTarget = p.height / 2 - targetY * zoomTarget;
			zoom = p.lerp(zoom, zoomTarget, 0.04)
			offsetX = p.lerp(offsetX, offsetXTarget, 0.04)
			offsetY = p.lerp(offsetY, offsetYTarget, 0.04)
		}

		// Function to check if mouse is over the p5.js canvas
		function isMouseOverCanvas() {
		    // Get the element under the mouse
		    let element = document.elementFromPoint(p.mouseX + p.canvas.offsetLeft, p.mouseY + p.canvas.offsetTop);
		    // Return true if the element is the p5.js canvas
		    return element === p.canvas;
		}

		// Handle zooming with mouse wheel
		p.mouseWheel = (event) => {
		    if (!isMouseOverCanvas()) return; // Prevent zooming if not over the canvas

		    // Adjust zoom speed based on the velocity of the scroll (event.delta)
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
		};

		// Handle panning with mouse drag
		p.mousePressed = () => {
		    if (!isMouseOverCanvas()) return; // Prevent panning if not over the canvas
		    startX = p.mouseX - offsetX;
		    startY = p.mouseY - offsetY;
		};

		p.mouseDragged = () => {
		    if (!isMouseOverCanvas()) return; // Prevent dragging if not over the canvas
		    offsetX = p.mouseX - startX;
		    offsetY = p.mouseY - startY;
		};

		// Handle pinch zoom and pan for touchscreens
		p.touchStarted = () => {
		    if (!isMouseOverCanvas()) return; // Prevent touch actions if not over the canvas
		    if (p.touches.length === 1) {
		        // Single touch for panning
		        startX = p.touches[0].x - offsetX;
		        startY = p.touches[0].y - offsetY;
		    } else if (p.touches.length === 2) {
		        // Store the initial positions of two touches for pinch zoom
		        previousDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);
		    }
		};

		p.touchMoved = () => {
		    if (!isMouseOverCanvas()) return false; // Prevent touch movement if not over the canvas

		    if (p.touches.length === 1) {
		        // Pan with single finger swipe
		        offsetX = p.touches[0].x - startX;
		        offsetY = p.touches[0].y - startY;
		    } else if (p.touches.length === 2) {
		        // Pinch to zoom with two fingers
		        let currentDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);

		        // Calculate the zoom change based on the distance between two touches
		        if (previousDistance) {
		            let zoomChange = currentDistance / previousDistance;
		            let previousZoom = zoom;
		            zoom = p.constrain(zoom * zoomChange, zoomMinMax[0], zoomMinMax[1]); // Constrain the zoom level

		            // Calculate midpoint between the two touch points
		            let midX = (p.touches[0].x + p.touches[1].x) / 2;
		            let midY = (p.touches[0].y + p.touches[1].y) / 2;

		            // Adjust offsets to maintain zoom centered on the pinch
		            offsetX = (midX - offsetX) * (1 - zoomChange) + offsetX;
		            offsetY = (midY - offsetY) * (1 - zoomChange) + offsetY;
		        }

		        // Update previous distance for the next move event
		        previousDistance = currentDistance;
		    }

		    return false; // Prevent default behavior (like scrolling the page)
		};

		p.touchEnded = () => {
		    // Reset after touch ends
		    previousDistance = null;
		};

		let resizeTimeout;

		p.windowResized = () => {
			resize();
		};

		function resize() {
			w = p.windowWidth;
			// h = p.windowHeight - 6;
			h = p.windowHeight - marginBottom;
			dotSize = 4;
			if (p.windowWidth < 1500) {
				dotSize = 3;
			}
			if (p.windowWidth < 600) {
				dotSize = 2;
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
				this.radius = this.calculateOptimalRadius(obj.dots, dotSize);
				this.currentColors = Array(this.obj.dots).fill(p.color("#523c50"));
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					-200
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.peoplePositions = this.calculatePeoplePositions(obj.dots);
				this.hovered = false;
				this.score = 0;
				this.textDisplayed = false;
				this.varPct = 0; // Initialize varPct
        		this.prevVarPct = 0; // Initialize previous varPct
			}

			calculateOptimalRadius(totalDots, dotSize) {
		        if (totalDots <= 1) {
			        return 1 * dotSize*3; // Only the center dot, hence 1 circle
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
		        let targetVarPct = Number(String(this.obj[currentVar]).replace(/[^0-9.]/g, ''));
		        this.varPct = p.lerp(this.varPct, targetVarPct, 0.1); // Gradually move towards the target value
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
			    // Set the target x and y positions based on data
			    this.target.x = p.constrain(p.map(data[this.index].score, minmax[0], minmax[1], 0, w), 0, w); // Adjust x based on score
			    this.target.y = p.constrain(p.map(data[this.index].A_MEAN, 20000, 180000, h, marginTop), marginTop, h); // Adjust y based on A_MEAN

			    // Calculate the direction to the target position for both x and y
			    let directionToTarget = p.createVector(this.target.x - this.center.x, this.target.y - this.center.y);
			    directionToTarget.mult(0.05); // Control the speed of movement toward the target

			    let friction = this.velocity.copy();
			    friction.mult(-0.2); // Apply a small amount of friction to smooth out the movement

			    // Apply movement toward the target and friction
			    this.acceleration.add(directionToTarget);
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
			    let minDist = (this.radius/1.5 + other.radius/1.5); // The minimum distance to prevent overlap
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
			        let bounceFactor = 0.4; // Further reduce bounce intensity
			        let bounceEffect = direction.copy().mult(bounce * bounceFactor);
			        this.velocity.sub(bounceEffect);
			        other.velocity.add(bounceEffect);

			        // Apply strong damping to reduce velocity after bounce
			        let dampingFactor = 0.2; // Increase damping to reduce energy retention further
			        this.velocity.mult(dampingFactor);
			        other.velocity.mult(dampingFactor);

			        // Gradually reduce velocity to encourage settling
			        this.velocity.mult(0.95);
			        other.velocity.mult(0.95);

			        // Stop the circles if their velocity is below a threshold
			        let velocityThreshold = 0.2; // Define a low velocity threshold
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

		        p.circle(this.center.x, this.center.y, this.radius);

		        const transitionSpeed = 0.3; // Smooth transition speed
		        const filledColor = p.color("#ff69f2");
		        const unfilledColor = p.color("#523c50");

		        // Calculate how many dots to fill based on smoothed varPct
		        const dotsToFill = Math.round(this.obj.dots * (this.varPct / 100));

		        // Display each dot with smooth color transitions
		        for (let i = 0; i < this.peoplePositions.length; i++) {
		            p.noStroke();

		            // Determine target color based on the smoothed fill state
		            let targetColor = i < dotsToFill ? filledColor : unfilledColor;

		            // Smoothly interpolate the current color towards the target color
		            this.currentColors[i] = p.lerpColor(this.currentColors[i], targetColor, transitionSpeed);

		            // Set the fill to the interpolated color
		            p.fill(this.currentColors[i]);
		            p.ellipse(this.peoplePositions[i].x, this.peoplePositions[i].y, dotSize, dotSize);
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

			   let shouldDisplayText = 
			    (this.radius > 30 && !this.checkTextOverlap(otherCircles)) || 
			    this.hovered || 
			    (!this.checkTextOverlap(otherCircles) && (guidedTour || zoomedGuidedTour));

			    if (searchValue != "" && searchValue != this.obj.OCCUPATION) {
			    	shouldDisplayText = false;
			    } else if (searchValue != "" && searchValue == this.obj.OCCUPATION) {
			    	shouldDisplayText = true;
			    }

			    const textWidth = p.textWidth(this.obj.OCC_SHORT);

			    if ( (this.center.x - textWidth/2 < 0 || this.center.x + textWidth/2 > w) && Math.abs(zoom - 1) < 0.1 ) {
			    	shouldDisplayText = false;
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
			        p.fill(255, 255, 255, this.alpha);

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
			        p.noStroke();
			        p.text(this.obj.OCC_SHORT, xPos, yPos);

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