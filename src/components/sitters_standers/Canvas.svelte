<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let data, currentData, questions, copy, currentQuestionNum, currentStageNumber, minmax, minIndicies, maxIndicies;

	let circles = [];
	const dotSize = 5; 
	export let currentVar;
	let new_currentVar = -1;
	let prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
	let multiplier = 0.038;
	let guidedTour = true;
	let zoomedGuidedTour = true;

	let zoom = 1;       // Zoom level
	let zoomTarget = 1;
	let zoomMinMax = [0.2, 6];
	let offsetX = 0;    // X offset for panning
	let offsetXTarget = 0;
	let offsetY = 0;    // Y offset for panning
	let offsetYTarget = 0;
	let startX, startY; // To store the starting position of dragging
	let startTouches = []; // To store touch points for pinch zoom
	let previousDistance = null; // To store the previous distance between two touches
	let atlasGrotesk;
	let dotDivider = 1;

	const sketch = (p) => {

		p.preload = () => {
			atlasGrotesk = p.loadFont('assets/app/AtlasGrotesk-Regular-Web.otf');
		}
		p.setup = () => {
			w = p.constrain(p.windowWidth,800,6000);
			h = p.windowHeight - 6;
			if (p.windowWidth < 800) {
				zoom = 0.45;
				// dotDivider = 2;
				h = p.windowHeight*2;
			}
			
			p.createCanvas(w, h);
			

			for (let i = 0; i < data.length; i++) {
				let circle = new Circle(data[i], i);
				circles.push(circle);
			}
			p.canvas.oncontextmenu = () => false;
		};

		p.draw = () => {
			setStage();
			p.clear();
			p.smooth();
			p.background("#211d21");
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
				// for (let i = 0; i < circles.length; i++) {
				// 	circles[i].updateGroup();
				// }
				prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
				new_currentVar = currentVar;
			}
			p.pop();
			// p.fill(255);
			// p.textAlign(p.LEFT);
			// p.text("Zoom: " + String(zoom.toFixed(2)), 10, 100);
			// p.text("Offset: [" + String(offsetX.toFixed(2)) + "," + String(offsetY.toFixed(2)) + "]", 10, 130);
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
				centerAndZoomOnCoordinate(circles[maxIndicies[0]].center.x, circles[maxIndicies[0]].center.y, 1);
			}
			if (copy.story[currentStageNumber].stage == "other_dissimilar_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, .9);
			}
			if (copy.story[currentStageNumber].stage == "all_jobs") {
				centerAndZoomOnCoordinate(w/2, h/2, .9);
			}
		}

		function centerAndZoomOnCoordinate(targetX, targetY, targetZoom) {
		    // Set the zoom level to the target zoom
			zoomTarget = targetZoom;

		    // // Calculate the offset to center the target coordinate on the canvas
			offsetXTarget = p.width / 2 - targetX * zoomTarget;
			offsetYTarget = p.height / 2 - targetY * zoomTarget;
			zoom = p.lerp(zoom, zoomTarget, 0.05)
			offsetX = p.lerp(offsetX, offsetXTarget, 0.05)
			offsetY = p.lerp(offsetY, offsetYTarget, 0.05)
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

            // Adjust the offset to keep zoom centered between the two touch points
            let midX = (p.touches[0].x + p.touches[1].x) / 2;
            let midY = (p.touches[0].y + p.touches[1].y) / 2;
            offsetX = midX - (midX - offsetX) * zoomChange;
            offsetY = midY - (midY - offsetY) * zoomChange;
        }

        // Update previous distance
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
				
				if (obj.dots/dotDivider <= 1) {
					this.radius = dotSize*3;
				} else if (obj.dots/dotDivider <= 7) {
					this.radius = dotSize*5;
				} else {
					this.radius = Math.sqrt(obj.TOT_EMP) * multiplier;
				}
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					-200
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.maxSpeed = 3;
				this.maxForce = 0.5;
				this.peoplePositions = this.calculatePeoplePositions(obj.dots/dotDivider);
				this.hovered = false;
				this.score = 0;
				this.textDisplayed = false;
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
			    // Set the target x and y positions based on data
			    this.target.x = p.map(data[this.index].score, minmax[0], minmax[1], 0, w); // Adjust x based on score
			    this.target.y = p.map(data[this.index].A_MEAN, 0, 200000, h, 0); // Adjust y based on A_MEAN

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
			        let bounceFactor = 0.2; // Further reduce bounce intensity
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
				p.strokeWeight(0.4/zoom);
				p.stroke("#947594");
				if (this.hovered || this.textDisplayed) {
					p.stroke("#ffffff")
					p.strokeWeight(0.4/zoom);
				}
				p.circle(this.center.x, this.center.y, this.radius);
				// Display each Person as a small circle

				const dotsToFill = Math.round(this.obj.dots/dotDivider * (this.varPct / 100));
				for (let i = 0; i < this.peoplePositions.length; i++) {
					p.noStroke();
					if (i < dotsToFill) {
						p.fill("#ff69f2"); // Fill color for dots that should be filled
					} else {
						p.fill("#523c50"); // Fill color for remaining dots
					}
					p.ellipse(this.peoplePositions[i].x, this.peoplePositions[i].y, dotSize, dotSize);
				}
				
			}

			checkTextOverlap(otherCircles) {
			    // Approximate text height and width based on zoom and font size
			    let scaledFontSize = h / 12 / zoom;
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

			   const shouldDisplayText = 
			    (this.radius > 30 && !this.checkTextOverlap(otherCircles)) || 
			    this.hovered || 
			    (!this.checkTextOverlap(otherCircles) && (guidedTour || zoomedGuidedTour));

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