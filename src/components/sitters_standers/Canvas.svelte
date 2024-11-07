<script>
	import P5 from 'p5-svelte';
	let w = 55;
	let h = 55;
	export let bg, data, currentData, questions, copy, currentQuestionNum, currentStageNumber, minmax, minIndicies, maxIndicies, searchValue, explored, x_axis_variable, x_axis_variable_range, currentVar, reset, axis_flip, axis_variable, prefersReducedMotion, showAverageStage;
	let prevStageNumber = currentStageNumber;
	let circles = [];
	let dotSize = 6; 
	let new_currentVar = -1;
	let prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
	let multiplier = 0.038;
	let guidedTour = true;
	let zoomedGuidedTour = true;
	let job_hl_index = 0;
	let userControl = false;
	let zoom = 4;       // Zoom level
	let zoomTarget = 4;
	let zoomMinMax = [0.5, 10];
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
	let stageSet = false;
	const gridSize = 100; // Size of each grid cell
	let grid = {}; // Spatial partitioning grid to track circle positions
	let highlightedJobs = [];
	let hlcolor = [0,0,0];
	let bgType = copy.story[currentStageNumber].bg;
	let standerGifs = [];
	let sitterGifs = [];
	const standerNum = 8;
	const sitterNum = 8;
	let hoveredCircle = null;

	const sketch = (p) => {
		let hlYellow = p.color(241, 82, 255);
		let labelPurple = p.color(189, 165, 196);

		p.preload = () => {
			atlasGrotesk = p.loadFont('assets/app/AtlasGrotesk-Regular-Web.otf'); // TiemposTextWeb-Regular.otf
			for (let i = 0; i < standerNum; i++) {
				standerGifs.push(p.loadImage('assets/app/Stander' + i + '.gif'));
			}
			for (let i = 0; i < sitterNum; i++) {
				sitterGifs.push(p.loadImage('assets/app/Sitter' + i + '.gif'));
			}
		}
		p.setup = () => {
			w = p.constrain(p.windowWidth,800,6000);
			h = p.windowHeight - marginBottom;
			dotSize = 6;
			p.createCanvas(w, h);
			

			for (let i = 0; i < data.length; i++) {
				let circle = new Circle(data[i], i);
				circles.push(circle);
			}
			p.canvas.oncontextmenu = () => false;
			resize();
		};

		function addToGrid(circle) {
			let gridX = Math.floor(circle.center.x / gridSize);
			let gridY = Math.floor(circle.center.y / gridSize);
			if (!grid[gridX]) grid[gridX] = {};
			if (!grid[gridX][gridY]) grid[gridX][gridY] = [];
			grid[gridX][gridY].push(circle);
		}

		function clearGrid() {
			grid = {};
		}

		p.draw = () => {

			if (reset) {
				p.reset();
				reset = false;
			}

			hlYellow.setAlpha(255);
			if (currentStageNumber != prevStageNumber || zoomedGuidedTour) {
				bgType = copy.story[currentStageNumber].bg;
				userControl = false;
				prevStageNumber = currentStageNumber;
				if (no_hl_list.indexOf(currentVar) != -1) {
					no_hl = true;
				} else {
					no_hl = false;
				}
				if (copy.story[currentStageNumber].job) {
					highlightedJobs = copy.story[currentStageNumber].job.split("|").concat(["Sitter avg.", "Stander avg."]);
				} else {
					highlightedJobs = [];
				}
			}

			if (!userControl) {
				setStage();
			}
			p.clear();
			p.noSmooth();
			p.background("#150317");

			clearGrid();
			p.push();
			p.translate(offsetX, offsetY);
			p.scale(zoom);

			// backgroundhl();
			axisLines();
			averageLine();
			for (let i = 0; i < circles.length; i++) {
			    if (!checkDisplay(i) || data[i].score == -1) continue;

			    circles[i].updateGroup();
			    circles[i].update();
			    circles[i].display();

			    if (prefersReducedMotion || ["Sitter avg.", "Stander avg."].includes(circles[i].obj.OCCUPATION)) continue;

			    const withinViewport = w > 860 || (
			        Math.abs(circles[i].center.y - circles[i].target.y) < h / 10 &&
			        Math.abs(circles[i].center.x - circles[i].target.x) < h / 10
			    );

			    if (!withinViewport) continue;

			    for (let j = 0; j < circles.length; j++) {
			        if (j === i || data[j].score === -1) continue;

			        const withinTargetRange = w > 860 || (
			            Math.abs(circles[i].target.y - circles[j].target.y) < h / 10 &&
			            Math.abs(circles[i].target.x - circles[j].target.x) < h / 10
			        );

			        if (withinTargetRange) {
			            circles[i].collide(circles[j]);
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
                		hoveredCircle = circles[i];
					}
					circles[i].displayText(circles);
				}
			}

			if (currentVar != new_currentVar) {
				prevVar = currentQuestionNum == 0 ? copy.questions[0].variable : copy.questions[currentQuestionNum - 1].variable;
				new_currentVar = currentVar;
				if (stage != "explore") {
					stageSet = false;
				}
			}
			
			axisLabels();
			p.pop();
		};

		function averageLine() {
			if (currentStageNumber >= showAverageStage) {
				// p.stroke(255, 0, 251,160);
				hlYellow.setAlpha(200);
				p.stroke(hlYellow);
				p.strokeWeight(0.8/zoom);
				p.line(circles[circles.length-2].center.x,circles[circles.length-2].center.y,circles[circles.length-3].center.x, circles[circles.length-3].center.y);
			}
		}

		function axisLines() {
			if (currentStageNumber !== 0) {
        		// Set common properties
				p.stroke("#452345");
				p.strokeWeight(1 / zoom);
				p.line(w / 2, -h * 100 * zoom, w / 2, h * 100 * zoom);
				p.line(-w * 100 * zoom, h / 2, w * 100 * zoom, h / 2);
			}
		}


		function axisLabels() {
			p.textSize(14/zoom);
			if (currentStageNumber !== 0) {
				p.fill("#eba800");
       			 // Function to set stroke color based on bgType
				const getStrokeColor = () => (bgType === "sit" || bgType === "stand" ? hlcolor : "#150317");

        		// Draw arrows and labels with adjusted alignment and stroke color
				p.stroke(getStrokeColor());
				p.textAlign(p.RIGHT);
				p.text("←", w / 2 - (79 / zoom), 23 / zoom);

				p.textAlign(p.LEFT);
				p.text("→", w / 2 + (90 / zoom), 23 / zoom);

				if (axis_variable !== -1) {
					p.text("↑", 0, h / 2 - (5 / zoom));
					p.text("↓", 0, h / 2 + (20 / zoom));
				}

        		// Set font for "Sitters" and "Standers" labels
				p.textFont(atlasGrotesk);
				p.textAlign(p.RIGHT);
				p.text("Sitters", w / 2 - (34 / zoom), 23 / zoom);

				p.textAlign(p.LEFT);
				p.text("Standers", w / 2 + (30 / zoom), 23 / zoom);

				if (axis_variable !== -1) {
					p.text("   More " + axis_variable, 3, h / 2 - (5 / zoom));
					p.text("   Less " + axis_variable, 3, h / 2 + (20 / zoom));
				}
			}
		}


	    let alphaValue = [0, 0]; // Initial alpha values for background highlights

	    function adjustAlpha(index, increment) {
	    	alphaValue[index] += increment ? 5 : -5;
		    alphaValue[index] = p.constrain(alphaValue[index], 0, 255); // Keep alpha within 0-255
		}

		function backgroundhl() {

		    // Adjust alpha for "sit" background
			adjustAlpha(0, bgType === "sit");
			if (bgType === "sit") {
				p.fill(hlcolor[0], hlcolor[1], hlcolor[2], alphaValue[0]);
				p.rect(w / 2, -h * 10, -w * 10, h * 100);
			}

		    // Adjust alpha for "stand" background
			adjustAlpha(1, bgType === "stand");
			if (bgType === "stand") {
				p.fill(hlcolor[0], hlcolor[1], hlcolor[2], alphaValue[1]);
				p.rect(w / 2, -h * 10, w * 10, h * 100);
			}
		}

		function checkDisplay(n) {
			zoomedGuidedTour = true;
			guidedTour = true;
			stage = copy.story[currentStageNumber].stage;
			if (stage == "other_similar_jobs_you" && (maxIndicies.includes(n) || n == data.length-1)) {
				return true;
			}
			if (currentStageNumber < showAverageStage && circles[n].obj.yellow && n != circles.length -1) {
				return false;
			}
			if (stage == "explore" || stage == "preexplore" || stage == "preexplore2") {
				return true;
			}
			if (stage == "one_similar_job" && n == maxIndicies[0]) {
				return true;
			}
			if (stage == "other_similar_jobs" && maxIndicies.includes(n)) {
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
				if (!stageSet) {
					searchValue = "";
					centerAndZoomOnCoordinate(w/2, h/2, 0.95, 1);
					stageSet = true;
				}
			}
			if (copy.story[currentStageNumber].stage == "preexplore" || copy.story[currentStageNumber].stage == "preexplore2") {
				centerAndZoomOnCoordinate(w/2, h/2, 0.95);
			}
			if (copy.story[currentStageNumber].stage == "one_similar_job") {
				centerAndZoomOnCoordinate(circles[maxIndicies[0]].center.x, circles[maxIndicies[0]].center.y, 10, 0.2);
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
				centerAndZoomOnCoordinate(w/2, h/2, 0.95);
			}
		}

		function centerAndZoomOnCoordinate(targetX, targetY, targetZoom, speed) {
		    // Set the zoom level to the target zoom
			zoomTarget = targetZoom;

		    // // Calculate the offset to center the target coordinate on the canvas
			offsetXTarget = p.width / 2 - targetX * zoomTarget;
			offsetYTarget = p.height / 2 - targetY * zoomTarget;
			let lerpSpeed = 0.05;
			if (speed != undefined) {
				lerpSpeed = speed;
			}
			maxSpeed = 8;
			maxForce = 100;
			if (currentStageNumber == 0) {
				maxSpeed = 15;
				maxForce = 100;
			} else if (zoomedGuidedTour || userControl) {
				lerpSpeed = 0.1;
				maxSpeed = 7;
				maxForce = 100;
			} 
			if (speed == 1 || prefersReducedMotion) {
				zoom = zoomTarget;
				offsetX = offsetXTarget; 
				offsetY = offsetYTarget;
			} else {
				zoom = p.lerp(zoom, zoomTarget, lerpSpeed);
				offsetX = p.lerp(offsetX, offsetXTarget, lerpSpeed);
				offsetY = p.lerp(offsetY, offsetYTarget, lerpSpeed);	
			}
		}

		// Function to check if mouse is over the p5.js canvas
		function isMouseOverCanvas() {
		    // Get the element under the mouse
			let element = document.elementFromPoint(p.mouseX + p.canvas.offsetLeft, p.mouseY + p.canvas.offsetTop);
		    // Return true if the element is the p5.js canvas
			return element === p.canvas;
		}
		let isDragging = false;
		let hasDragged = false;

		// Handle zooming with mouse wheel
	p.mouseWheel = (event) => {
    if (!isMouseOverCanvas()) return;
    userControl = true;
    explored = true;

    let scrollVelocity = Math.abs(event.delta);
    let baseZoomSpeed = 0.05 * zoom;
    let zoomSpeed = baseZoomSpeed * (scrollVelocity / 100);

    let previousZoom = zoom;

    if (event.delta > 0 && zoom > zoomMinMax[0]) {
        zoom = p.constrain(zoom - zoomSpeed, zoomMinMax[0], zoomMinMax[1]);
    } else if (event.delta < 0 && zoom < zoomMinMax[1]) {
        zoom = p.constrain(zoom + zoomSpeed, zoomMinMax[0], zoomMinMax[1]);
    }

    let zoomChange = zoom / previousZoom;

    offsetX = p.mouseX - (p.mouseX - offsetX) * zoomChange;
    offsetY = p.mouseY - (p.mouseY - offsetY) * zoomChange;
};


		// Handle panning with mouse drag
		p.mousePressed = () => {
    if (!isMouseOverCanvas()) return;

    startX = p.mouseX - offsetX;
    startY = p.mouseY - offsetY;
    isDragging = true; // Start dragging
    hasDragged = false; // Reset drag flag
    hoveredCircle = null;
};

p.mouseDragged = () => {
    if (!isMouseOverCanvas()) return;
    userControl = true;
    explored = true;

    offsetX = p.mouseX - startX;
    offsetY = p.mouseY - startY;
    hasDragged = true; // Mark that dragging occurred
};

p.mouseReleased = () => {
    if (!isDragging) return;

    isDragging = false; // Stop dragging
    if (!hasDragged && hoveredCircle && copy.story[currentStageNumber].stage === "explore") {
        searchValue = hoveredCircle.obj.OCCUPATION; // Only update if no dragging occurred
    }
    hoveredCircle = null;
};


		let isTouchDragging = false;
let hasTouchDragged = false;
let pinchZooming = false;
let wasPinching = false;

// Handle touch start
p.touchStarted = () => {
    userControl = true;

    if (!isMouseOverCanvas()) return;

    if (p.touches.length === 1 && !pinchZooming) {
        // Single touch for panning
        startX = p.touches[0].x - offsetX;
        startY = p.touches[0].y - offsetY;
        isTouchDragging = true;   // Start dragging
        hasTouchDragged = false;  // Reset drag flag
    } else if (p.touches.length === 2) {
        // Two touches for pinch zoom
        pinchZooming = true;
        wasPinching = true;
        explored = true;
        previousDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);
    }
};

// Handle touch move (dragging or pinch zoom)
p.touchMoved = () => {
    if (!isMouseOverCanvas()) return false;
    userControl = true;

    if (p.touches.length === 1) {
        if (pinchZooming) {
            startX = p.touches[0].x - offsetX;
            startY = p.touches[0].y - offsetY;
            pinchZooming = false; // Exit pinch zoom mode
        }
        // Pan with single finger swipe
        offsetX = p.touches[0].x - startX;
        offsetY = p.touches[0].y - startY;
        hasTouchDragged = true; // Mark as dragging occurred
    } else if (p.touches.length === 2) {
        let currentDistance = p.dist(p.touches[0].x, p.touches[0].y, p.touches[1].x, p.touches[1].y);

        if (previousDistance) {
            let zoomChange = currentDistance / previousDistance;
            let newZoom = zoom * zoomChange;

            // Only allow zooming within constraints
            if (newZoom >= zoomMinMax[0] && newZoom <= zoomMinMax[1]) {
                zoom = newZoom;

                let midX = (p.touches[0].x + p.touches[1].x) / 2;
                let midY = (p.touches[0].y + p.touches[1].y) / 2;

                offsetX = (midX - offsetX) * (1 - zoomChange) + offsetX;
                offsetY = (midY - offsetY) * (1 - zoomChange) + offsetY;
            }
        }

        previousDistance = currentDistance;
        pinchZooming = true;
    }

    return false; // Prevent default behavior
};

// Handle touch end
p.touchEnded = () => {
    userControl = true;

    if (p.touches.length === 1 && wasPinching) {
        startX = p.touches[0].x - offsetX;
        startY = p.touches[0].y - offsetY;
        wasPinching = false;
    }

    if (!hasTouchDragged && !pinchZooming && hoveredCircle && copy.story[currentStageNumber].stage === "explore") {
        searchValue = hoveredCircle.obj.OCCUPATION; // Only update if no dragging or pinch zoom occurred
    }

    // Reset flags after touch ends
    isTouchDragging = false;
    hasTouchDragged = false;
    previousDistance = null;
    hoveredCircle = null;
};

		let resizeTimeout;

		p.windowResized = () => {
			resize();
		};

		function resize() {
			if (p.windowWidth < 860) {
				w = p.windowWidth;
				h = p.windowHeight - marginBottom + 20;
				marginTop = 70;
			} else {
				w = p.windowWidth - 370;
				h = p.windowHeight;
				marginTop = 60;
			}
			
			dotSize = 7;
			if (p.windowWidth < 1500) {
				dotSize = 6;
			}
			if (p.windowWidth < 860) {
				dotSize = 5;
			}
			if (p.windowWidth < 500) {
				dotSize = w/150;
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
				this.radius = this.calculateOptimalSquareDimensions(obj.dots / divider, dotSize);
				if (obj.OCCUPATION == "You" || obj.OCCUPATION == "Sitter avg." || obj.OCCUPATION == "Stander avg.") {
					this.radius = 30;
				}
				this.currentColors = Array(this.obj.dots).fill(p.color("#523c50"));
				this.center = p.createVector(
					p.random(this.radius, w - this.radius),
					p.random(-10,-h),
					);
				this.velocity = p.createVector(0, 0);
				this.acceleration = p.createVector(0, 0);
				this.target = this.center.copy();
				this.peoplePositions = this.calculatePeoplePositions(obj.dots / divider);
				this.hovered = false;
				this.maxSpeed = 1;
				this.maxForce = 1;
				this.score = 0;
				this.textDisplayed = false;
				this.varPct = 0; // Initialize varPct
				this.targetVarPct = 0;
        		this.prevVarPct = 0; // Initialize previous varPct
        		this.xValue = obj[x_axis_variable];
        		this.personTypeStander = [];
        		for (let i = 0; i < this.peoplePositions.length; i++) {
        			this.personTypeStander.push( Math.floor(Math.random() * standerNum));
        		}
        		this.personTypeSitter = [];
        		for (let i = 0; i < this.peoplePositions.length; i++) {
        			this.personTypeSitter.push( Math.floor(Math.random() * sitterNum));
        		}
        	}

        	calculateOptimalSquareDimensions(totalDots, dotSize) {
        		if (totalDots == 1) {
        			totalDots = 1.2;
        		}
			    let gridSize = Math.ceil(Math.sqrt(totalDots)); // Number of dots per row/column
			    let spacing = dotSize * 0.8; // Tighter spacing between dots

			    // Calculate the total width and height of the grid
			    let width = (gridSize - 1) * spacing + dotSize; // Total width of the grid
			    let height = (gridSize - 1) * spacing + dotSize; // Total height of the grid

			    // Since we want a square, take the larger of the width and height
			    let squareSize = Math.max(height, height);

			    return squareSize; // Return the size of the square's side
			}

			calculatePeoplePositions(totalPeople) {
				let positions = [];
				let centerX = this.center.x;
				let centerY = this.center.y;
			    let gridSize = Math.ceil(Math.sqrt(totalPeople)); // Number of dots per row/column
			    let spacing = dotSize * 0.6; // Tighter spacing between dots on both axes

			    let startX = centerX - ((gridSize - 1) / 2) * spacing; // Center the grid horizontally
			    let startY = centerY - ((gridSize - 1) / 2) * spacing; // Center the grid vertically

			    let dotsPlaced = 0;
			    for (let row = 0; row < gridSize && dotsPlaced < totalPeople; row++) {
			    	for (let col = 0; col < gridSize && dotsPlaced < totalPeople; col++) {
			    		let x = startX + col * spacing;
			    		let y = startY + row * spacing;
			    		positions.push(p.createVector(x, y));
			    		dotsPlaced++;
			    	}
			    }

			    return positions;
			}


			updatePeoplePositions(newCenter) {
			    let gridSize = Math.ceil(Math.sqrt(this.peoplePositions.length));
			    let spacing = dotSize * 0.6; // Tighter spacing between dots

			    let startX = newCenter.x - ((gridSize - 1) / 2) * spacing; // Center the grid horizontally
			    let startY = newCenter.y - ((gridSize - 1) / 2) * spacing; // Center the grid vertically

			    if (gridSize == 1) {
			    	let startY = newCenter.y;
			    }

			    // Check if there are more columns than rows and adjust startY
			    let totalRows = Math.ceil(this.peoplePositions.length / gridSize);
			    if (gridSize > totalRows) {
			        startY += dotSize/2.8; // Move positions up by half a grid size
			    }

			    let dotsPlaced = 0;
			    for (let row = 0; row < gridSize && dotsPlaced < this.peoplePositions.length; row++) {
			        for (let col = 0; col < gridSize && dotsPlaced < this.peoplePositions.length; col++) {
			            let x = startX + col * spacing;
			            let y = startY + row * spacing;
			            this.peoplePositions[dotsPlaced] = p.createVector(x, y);
			            dotsPlaced++;
			        }
			    }
			}

			updateGroup() {
				this.score = 0;
				for (let i = 0; i < currentQuestionNum; i++) {
					const question = questions[i];
					let value = String(this.obj[question]).replace(/[^0-9.]/g, '');
					if (value !== "" && Number(value) > 0) {
						if (Number(value) > Number(copy.questions[i].threshold) && currentData[i] === 0 && value > 0) {
							this.score = 1;
						}
					}
				}

		        // Smoothly transition varPct using lerp for a smoother visual effect
				this.targetVarPct = Number(String(this.obj[currentVar]).replace(/[^0-9.]/g, ''));
				if (copy.story[currentStageNumber].stage == "preexplore2") {
					this.targetVarPct = 200;
				} else if (currentVar == "A_MEAN" || currentVar == "INJURY_RATE" || currentVar == undefined) {
					// this.varPct = 0;
					this.targetVarPct = 0;
				}
				
				if (prefersReducedMotion) {
					this.varPct = this.targetVarPct;
				} else {
					this.varPct = p.lerp(this.varPct, this.targetVarPct, 0.1); // Gradually move towards the target value
				}

			}


			update() {
				this.maxSpeed = maxSpeed;
				this.maxForce = maxForce;
				this.radius = this.calculateOptimalSquareDimensions(this.obj.dots / divider, dotSize);
				this.peoplePositions = this.calculatePeoplePositions(this.obj.dots / divider);

				if (this.obj.OCCUPATION == "You") {
					this.radius = 30;
				}
				if (this.obj.OCCUPATION == "Sitter avg." || this.obj.OCCUPATION == "Stander avg.") {
					this.radius = 30;
				}

			    // Calculate the main target position
				let targetY = w > 860
				? p.map(data[this.index].score, minmax[0], minmax[1], h - 50, marginTop)
				: p.map(data[this.index].score, minmax[0], minmax[1], h - 50, marginTop);

				if (axis_flip) {
					targetY = w > 860
					? p.map(data[this.index].score, minmax[0], minmax[1], marginTop, h - 50)
					: p.map(data[this.index].score, minmax[0], minmax[1], marginTop, h - 50);
				}


				this.target.y = p.constrain(targetY, marginTop, h);
				this.target.x = p.constrain(
					p.map(data[this.index][x_axis_variable], x_axis_variable_range[x_axis_variable][0], x_axis_variable_range[x_axis_variable][1], paddingX, w - paddingX),
					paddingX,
					w - paddingX
					);

			    // Adjust targets to avoid overlap
				this.adjustTargets();

			    // Calculate distance to adjusted target
				let distanceToTarget = p.dist(this.center.x, this.center.y, this.target.x, this.target.y);

			    // Easing effect with stronger damping near the target
				let easeThreshold = 10;
				if (prefersReducedMotion) {
					this.center.x = this.target.x;
					this.center.y = this.target.y;
				} else if (distanceToTarget < easeThreshold) {
			        // Gradually ease towards the target with increased speed
			        this.center.x = p.lerp(this.center.x, this.target.x, 0.1); // Increase factor for faster easing
			        this.center.y = p.lerp(this.center.y, this.target.y, 0.1);
			    } else {
			        // Normal movement towards the target when farther away
			    	let desired = p.createVector(this.target.x - this.center.x, this.target.y - this.center.y);
			    	let adjustedMaxSpeed = p.map(distanceToTarget, 0, 100, 0.1, this.maxSpeed);
			    	let adjustedMaxForce = p.map(distanceToTarget, 0, 100, 0.05, this.maxForce);
			    	desired.setMag(adjustedMaxSpeed);

			    	let steer = p.Vector.sub(desired, this.velocity);
			    	steer.limit(adjustedMaxForce);

			    	this.acceleration.add(steer);

			        // Update velocity with acceleration and apply dynamic damping
			    	this.velocity.add(this.acceleration);
			    	this.velocity.limit(this.maxSpeed);

			        let dampingFactor = p.map(distanceToTarget, easeThreshold, 100, 0.6, 0.9); // Increase damping closer to target
			        this.velocity.mult(dampingFactor);

			        // Update center position
			        this.center.add(this.velocity);
			    }

			    // Constrain within screen bounds
			    this.center.x = p.constrain(this.center.x, this.radius, w - this.radius);
			    this.center.y = p.constrain(this.center.y, this.radius + marginTop, h - this.radius);

			    this.acceleration.mult(0);

			    // Stop if velocity is very low to prevent minor jittering
			    this.checkStopThreshold();
			    this.updatePeoplePositions(this.center);
			}





			adjustTargets() {
			    let targetAdjustmentStrength = 0.2; // Gentle strength for subtle target adjustment

			    for (let other of circles) {
			    	if (other !== this) {
			    		let targetDistance = p.dist(this.target.x, this.target.y, other.target.x, other.target.y);

			            // Apply a gentle adjustment if targets are very close
			            if (targetDistance < this.radius) { // Adjust based on your desired spread
			            	let repulsion = p.createVector(this.target.x - other.target.x, this.target.y - other.target.y);
			            	repulsion.normalize();
			            	repulsion.mult(targetAdjustmentStrength);

			                this.target.add(repulsion); // Gently nudge target apart
			                other.target.sub(repulsion); // Counter-adjust other’s target
			            }
			        }
			    }
			}




			checkStopThreshold() {
			    let velocityThreshold = 0.01; // Lower value for quicker stop
			    if (this.velocity.mag() < velocityThreshold) {
			    	this.velocity.set(0, 0);
			    }
			}


			collide(other) {
				let distance = p.Vector.dist(this.center, other.center);
			    let minDist = (this.radius/1.5 + other.radius/1.5); // Minimum distance to prevent overlap

			    if (distance < minDist && !this.obj.yellow && !other.obj.yellow) {
			    	let overlap = minDist - distance;
			    	let direction = p.Vector.sub(other.center, this.center);
			    	direction.normalize();

			        // Apply a gentle correction to separate the circles
			        let correction = direction.copy().mult(overlap / 5); // Softer push for less jitter
			        this.center.sub(correction);
			        other.center.add(correction);

			        // Reduce bounce effect significantly to avoid rebounding
			        let relativeVelocity = p.Vector.sub(this.velocity, other.velocity);
			        let bounce = relativeVelocity.dot(direction);
			        let bounceFactor = 0.01; // Minimal bounce for gentler separation
			        let bounceEffect = direction.copy().mult(bounce * bounceFactor);
			        this.velocity.sub(bounceEffect);
			        other.velocity.add(bounceEffect);

			        // Strong damping to reduce jitter
			        this.velocity.mult(0.01);
			        other.velocity.mult(0.01);

			        // Gradually adjust target positions to reduce overlap
			        let targetOffset = 0.1; // Small offset to reduce overcrowding
			        this.target.add(p.createVector(p.random(-targetOffset, targetOffset), p.random(-targetOffset, targetOffset)));
			        other.target.add(p.createVector(p.random(-targetOffset, targetOffset), p.random(-targetOffset, targetOffset)));
			    }
			}




			
			display() {
				p.noFill();
				p.ellipseMode(p.CENTER);
				p.strokeWeight(0.4 / zoom);

				p.stroke("#5e485e");


				if ( this.hovered || (this.textDisplayed && stage != "explore")) {
					p.stroke("#d184d1");
					p.strokeWeight(1 / zoom);
				}

				if (searchValue == this.obj.OCCUPATION || highlightedJobs.indexOf(this.obj.OCCUPATION) != -1) {
					job_hl_index = this.index;
					p.stroke("#d184d1");
					p.strokeWeight(1 / zoom);
				}

				if (this.obj.OCCUPATION == "You" || this.obj.OCCUPATION == "Sitter avg." || this.obj.OCCUPATION == "Stander avg.") {
					hlYellow.setAlpha(255);
					p.stroke(hlYellow);
					p.fill(hlYellow);
					p.strokeWeight(0.4 / zoom);
					p.circle(this.center.x, this.center.y, 10);
				} else if (data[this.index].score != -1) {
					if (searchValue == this.obj.OCCUPATION || highlightedJobs.indexOf(this.obj.OCCUPATION) != -1) {
						// p.stroke(hlYellow);
						p.strokeWeight(1 / zoom);
					}

					p.square(this.center.x - this.radius/2, this.center.y - this.radius/2, this.radius, this.radius/3, this.radius/3, this.radius/3, this.radius/3);	
			        const transitionSpeed = 0.3; // Smooth transition speed
			        const filledColor = p.color("#ff69f2");
			        const unfilledColor = p.color("#523c50");

					// Calculate how many dots to fill based on smoothed varPct
			        let dotsToFill = this.obj.dots / divider * (this.varPct / 100);

					// Display each dot with smooth color transitions
			        for (let i = 0; i < this.peoplePositions.length; i++) {
			        	p.noStroke();

					    // Set the fill to unfilledColor to draw the background circle first
			        	p.fill(unfilledColor);
			        	if (i < Math.floor(dotsToFill)) {
			        		p.image(standerGifs[this.personTypeStander[i]], this.peoplePositions[i].x - dotSize/2, this.peoplePositions[i].y - dotSize/1.3, dotSize, dotSize);
			        	} else {
			        		p.image(sitterGifs[this.personTypeSitter[i]], this.peoplePositions[i].x - dotSize/2, this.peoplePositions[i].y - dotSize/1.3, dotSize, dotSize);
			        	}
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
			    const fadeSpeed = 20; // Slower fade speed for smoother transition
			    const overlapThreshold = 5; // Minimum overlap before hiding text

			    // Initialize alpha if not already defined
			    if (this.alpha === undefined) {
			        this.alpha = 0; // Start with fully transparent text
			    }

			    // Determine if prioritization should occur based on currentVar and varPct
			    let prioritizeVarPct = false;
			    const varPctThreshold = 20; // Threshold for varPct when currentVar is not empty

			    if (this.obj[currentVar] > 30 && (bg == "all") || ((this.xValue > minmax[1] / 2 && bg == "stand") || (this.xValue < minmax[1] / 2 && bg == "sit"))) {
			        prioritizeVarPct = true;
			    } else if (!currentVar || this.hovered) {
			        prioritizeVarPct = true;
			    }

			    // Always show text if hovered, regardless of searchValue
			    let shouldDisplayText = this.hovered || 
			        (searchValue !== "" && searchValue === this.obj.OCCUPATION) || 
			        this.obj.yellow ||
			        highlightedJobs.indexOf(this.obj.OCCUPATION) !== -1 || 
			        (prioritizeVarPct && (
			            (this.radius > 10 && !this.checkTextOverlap(otherCircles)) || 
			            (!this.checkTextOverlap(otherCircles) && (guidedTour || zoomedGuidedTour))
			        ));

			    if (shouldDisplayText) {
			        if (this.alpha < maxAlpha - overlapThreshold) {
			            this.alpha = Math.min(this.alpha + fadeSpeed, maxAlpha); // Fade in
			        }
			    } else {
			        if (this.alpha > overlapThreshold) {
			            this.alpha = Math.max(this.alpha - fadeSpeed, 0); // Fade out more gradually
			        } else {
			            this.alpha = 0; // Set to fully transparent if too low
			        }
			    }

			    // Only draw text if it's at least partially visible
			    if (this.alpha > 0) {
			    	if (searchValue === this.obj.OCCUPATION) {
			    		p.fill("#ffffff");
			    	} else if (this.obj.yellow) {
			            hlYellow.setAlpha(this.alpha);
			            p.fill(hlYellow);
			        } else {
			            labelPurple.setAlpha(this.alpha);
			            p.fill(labelPurple);
			        }

			        const maxTextWidth = 100 / zoom;
			        const scaledFontSize = 12 / zoom;
			        p.textSize(scaledFontSize);
			        p.textAlign(p.CENTER, p.BOTTOM);

			        let xPos = this.center.x;
			        let yPos = this.center.y - this.radius / 2;
			        if (this.obj.yellow) {
			             yPos = this.center.y - this.radius / 4;
			        }
			        p.stroke("#150317");
			        p.smooth();
			        p.strokeWeight(5 / zoom);

			        // Display text regardless of searchValue if hovered
			        if (this.hovered || searchValue === "" || this.obj.yellow || (searchValue === this.obj.OCCUPATION && data[this.index].score !== -1) || highlightedJobs.indexOf(this.obj.OCCUPATION) !== -1) {
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