<template>
  <!-- // create stage in vue konva -->
  <div class="container">
    <!-- // toolbar contain button stop and run and machine and queue -->
    <div class="toolbar">
      <button class="btn btn-primary" @click="runSimulation()">Run</button>
      <button class="btn btn-primary" @click="endSimulation()">Stop</button>
      <button class="btn btn-primary" @click="replay()">Replay</button>
      <button class="btn btn-primary" @click="resetGraph()">Reset</button>
      <button class="btn btn-primary" @click="saveImage()">Save Image</button>

      <button class="btn btn-primary" @click="addShape('circle')">Machine</button>
      <button class="btn btn-primary" @click="addShape('rect')">Queue</button>
      <button class="btn btn-primary" @click="startConnect()">Connect</button>
      <button class="btn btn-primary" @click="startDeleting()">Delete</button>
    </div>
    <!-- // draw contain stage and layer -->

    <!-- <h4>{{ selectedShapes }}</h4> -->
    <h4 style="color: red;">{{ errorMessage }}</h4>

    <div
      ref="stage"
      class="draw"
      @click="connectState > 0 ? handleStageClick() : null"
    ></div>
  </div>
</template>

<script>
import { Container } from "konva/lib/Container";
import Konva from "konva";

export default {
  components: {
    Container,
  },

  data() {
    return {
      stage: null,
      layer: null,
      connectState: 0, // 0 = not connecting, 1 = connecting first shape, 2 = connecting second shape, 3 = deleting
      selectedShapes: [],
      selectedShape: false,

      qCounter: 0,
      mCounter: 0,

      errorMessage: '',

      isRunning: false
    };
  },

  mounted() {
    this.stage = new Konva.Stage({
      container: this.$refs.stage,
      width: window.innerWidth,
      height: window.innerHeight,
    });

    this.layer = new Konva.Layer();
    this.stage.add(this.layer);

    this.stage.on('pointerover', (evt)=>{
      if (evt.target instanceof Konva.Shape && (this.selectedShape || this.connectState==3)) {
        this.stage.container().style.cursor = 'pointer'
        evt.target.strokeWidth(3)
      }
    })

    this.stage.on('pointerleave', (evt)=>{
      this.stage.container().style.cursor = 'default'
      if (evt.target instanceof Konva.Shape && (this.selectedShape || this.connectState==3)) {
        evt.target.strokeWidth(2)
      }
    })
  },

  methods: {

    dowloadURL(uri,name){

      var link=document.createElement('a');
      link.download=name;
      link.href=uri;
      document.body.appendChild(link);
      link.click()
      document.body.removeChild(link);
    


    },
    saveImage(){
      var dataURL=this.stage.toDataURL();
      this.dowloadURL(dataURL,"stage.png");


    },
    addRect() {
      const rect = new Konva.Rect({
        x: 50,
        y: 60,
        width: 70,
        height: 70,
        fill: "blue",
        stroke: "black",
        strokeWidth: 2,
        draggable: true,
        id: 'Q'+this.qCounter++,
        fromArrows: [],
        toArrows: [],
        text: null
      });

      let text = this.createText(rect)
      this.layer.add(rect);
      this.layer.add(text);
      this.layer.draw();
    },

    addCircle() {
      const circle = new Konva.Circle({
        x: 500,
        y: 50,
        radius: 30,
        fill: "green",
        stroke: "black",
        strokeWidth: 2,
        draggable: true,
        id: 'M'+this.mCounter++,
        fromArrows: [],
        toArrows: [],
        text: null
      });

      let text = this.createText(circle)
      this.layer.add(circle);
      this.layer.add(text)
      this.layer.draw();
    },

    addShape(button_type) {
      if (button_type == "circle") {
        this.addCircle();
      } else if (button_type == "rect") {
        this.addRect();
      } else {
        console.log("error");
      }
    },

    createText(shape) {
      let offset = shape.constructor.name=='Circle'? {x: shape.width() / 2, y: shape.height() / 2} : {x:0, y:0}
      var text = new Konva.Text({
        x: shape.x(),
        y: shape.y(),
        id: 'T'+shape.id(),
        text: shape.id() ,
        fontSize: 20,
        fontFamily: 'Calibri',
        width: shape.width(),
        padding: 10,
        height: shape.height(),
        align: 'center',
        draggable: true,
        verticalAlign: 'middle',
        offset: offset,
        listening: false
      });

      shape.attrs.text = text

      text.on('dragmove', ()=> {
        shape.x(text.x())
        shape.y(text.y())
      })

      shape.on('dragmove', ()=> {
        text.x(shape.x())
        text.y(shape.y())
      })

      return text
    },

    // create line to connect between them but points are at mouse click

    startConnect() {
      if (this.selectedShape==true) {
        const [shape1, shape2] = this.selectedShapes;
        shape1.strokeWidth(2)
        shape2.strokeWidth(2)
        this.connectState = 0
        this.selectedShapes = []
        this.selectedShape = false
      }
      else {
        this.connectState = 1; // Set to connecting first shape state
        this.selectedShape = true
      }
    },

    connectShapes() {
    
      if (this.connectState === 2 && this.selectedShapes.length === 2) {
        const [shape1, shape2] = this.selectedShapes;
        shape1.strokeWidth(2)
        shape2.strokeWidth(2)
        const arrow = new Konva.Arrow({
          points: [this.calcX(shape1,shape2), this.calcY(shape1,shape2), this.calcX(shape2,shape1), this.calcY(shape2,shape1)],
          pointerLength: 10,
          pointerWidth: 10,
          fill: 'black',
          stroke: 'black',
          strokeWidth: 2,
          fromVertex: shape1,
          toVertex: shape2
        });

        
        shape1.attrs.toArrows.push(arrow)
        shape2.attrs.fromArrows.push(arrow)

        shape1.on('dragmove', ()=> {
          arrow.points([this.calcX(shape1,shape2), this.calcY(shape1,shape2), this.calcX(shape2,shape1), this.calcY(shape2,shape1)])
          this.layer.batchDraw()
        })

        shape2.on('dragmove', ()=> {
          arrow.points([this.calcX(shape1,shape2), this.calcY(shape1,shape2), this.calcX(shape2,shape1), this.calcY(shape2,shape1)])
          this.layer.batchDraw()
        })
        
        this.layer.add(arrow);
        this.layer.draw();
        this.selectedShapes = [];
        this.connectState = 0; // Reset to not connecting state
        this.selectedShape = false
      } else {
        console.error("Select the second shape before connecting.");
      }
    },

    handleStageClick(event) {
      // Get the mouse coordinates on the stage
      const position = this.stage.getPointerPosition();

      // Try to find a shape at the clicked position
      const shape = this.stage.getIntersection(position);

      if (shape) {
        if (this.connectState === 2 && (shape.constructor.name==this.selectedShapes[0].constructor.name)) return
        

        // If we are in the connecting first shape state, move to the next state
        if (this.connectState === 1 && !(shape instanceof Konva.Arrow)) {
          this.selectedShapes.push(shape);
          this.connectState = 2;
        }
        // If we are in the connecting second shape state, connect the shapes
        else if (this.connectState === 2 && !(shape instanceof Konva.Arrow)) {
          this.selectedShapes.push(shape);
          this.connectShapes();
        }
        else if (this.connectState === 3) {
          this.deleteShape(shape)
        }
      }
    },

    calcX(shape1, shape2) {
      let x
      if (shape1.constructor.name=='Circle') {
        let dx = shape2.x() - shape1.x();
        let dy = shape2.y() - shape1.y();
        let angle = Math.atan2(dy, dx);
        let radius = shape1.radius();

        x = shape1.x() + radius * Math.cos(angle);
      }
      else {
        let dx = shape2.x() - shape1.x();
        let dy = shape2.y() - shape1.y();
        let angle = Math.atan2(dy, dx);
        x = shape1.x() + shape1.width() / 2 + shape1.width() / 2 * Math.cos(angle);
      }
      return x
    },

    calcY(shape1, shape2) {
      let y
      if (shape1.constructor.name=='Circle') {
        let dx = shape2.x() - shape1.x();
        let dy = shape2.y() - shape1.y();
        let angle = Math.atan2(dy, dx);
        let radius = shape1.radius();

        y = shape1.y() + radius * Math.sin(angle);
      }
      else {
        let dx = shape2.x() - shape1.x();
        let dy = shape2.y() - shape1.y();
        let angle = Math.atan2(dy, dx);
        y = shape1.y() + shape1.height() / 2 + shape1.height() / 2 * Math.sin(angle);
      }
      return y
    },

    createRunRequest() {
      let ms = this.stage.find('Circle')
      let from = []
      let to = []
      for (let m of ms) {
        let temp = []
        for (let arrow of m.attrs.fromArrows) {
          temp.push(arrow.attrs.fromVertex.id())
        }
        from.push(temp)
        to.push(m.attrs.toArrows[0].attrs.toVertex.id()) 
      }
      ms = ms.map((item)=> item.id())
      return {ms: ms, from: from, to: to}
    },

    runSimulation() {
      if (this.isRunning) {
        this.endSimulation()
      }
      if (this.validGraph()) {
        fetch(`http://localhost:8080/run`, {
          method: 'POST',
          body: JSON.stringify(this.createRunRequest()),
          headers: {
            'Content-Type' : 'application/json'
          }
        })
      .catch((e)=> console.log(e))
      }
      else {
        return
      }

      this.isRunning = true
      for (let text of this.stage.find('Text')) {
        if(text.id().includes('Q')) {
          text.text(text.text()+'\n'+0)
        }
      }
      this.update()
    },

    validGraph() {
      let isValid = true
      let machines = this.stage.find('Circle')
      for (let m of machines) {
        if (m.attrs.toArrows.length!=1 ||m.attrs.fromArrows.length<1) {
          isValid = false
          this.errorMessage = 'Current queuing network is not valid.'
          setTimeout(()=> this.errorMessage='', 5000)
        }else {
          for (let arrow of m.attrs.toArrows[0].attrs.toVertex.attrs.toArrows){
            if (arrow.attrs.toVertex==m) {
              isValid = false
              this.errorMessage = 'Current queuing network is not valid.'
              setTimeout(()=> this.errorMessage='', 5000)
            }
          }
        }
      }
      let initialQs = []
      let finalQs = []
      let queues = this.stage.find('Rect')
      for (let q of queues) {
        if (q.attrs.fromArrows.length==0) initialQs.push(q)
        if (q.attrs.toArrows.length==0) finalQs.push(q)
      }
      if (initialQs.length!=1 || finalQs.length!=1 || (initialQs.length==1 && initialQs[0]==finalQs[0])) {
        isValid = false
        this.errorMessage = 'Current queuing network is not valid.'
        setTimeout(()=> this.errorMessage='', 5000)
      }
      return isValid
    },

    update() {
      fetch(`http://localhost:8080/update`)
      .then(res=> res.json())
      .then((updateObj)=>{
        if (updateObj.mid!=='null') { 
          if (updateObj.mid==='end') {
            this.isRunning = false
            for (let text of this.stage.find('Text')) {
              if(text.id().includes('Q')) {
                text.text(text.text().split('\n')[0])
              }
            }
            for (let m of this.stage.find('Circle')) {
              m.fill('green')
            }
            this.connectState = 0
            this.selectedShapes = []
            this.selectedShape = false
            this.errorMessage = ''
            return
          }
          updateObj.color==null? this.stage.find('#'+updateObj.mid)[0].fill('green') : this.stage.find('#'+updateObj.mid)[0].fill(updateObj.color)
          this.stage.find('#T'+updateObj.qid)[0].text(updateObj.qid+'\n'+updateObj.size)
        }
      })
      .then(()=> {
        if(this.isRunning) {
          this.update()
        }
      })
      .catch((e)=> console.log(e))      
    },

    startDeleting() {
      if (this.connectState==3) {
        this.connectState = 0         
        this.stage.container().style.cursor = 'default'
      }
      else {
        this.connectState = 3
        this.selectedShapes = []
        this.selectedShape = false
      }
    },

    deleteShape(shape) {
      if (shape instanceof Konva.Arrow) {
        this.deleteArrow(shape)
      }
      else if (shape instanceof Konva.Circle) {
        this.deleteMachine(shape)
      }else if(shape instanceof Konva.Rect) {
        this.deleteQueue(shape)
      }
      this.connectState = 0 
      this.stage.container().style.cursor = 'default'
    },

    deleteArrow(arrow) {
      arrow.attrs.fromVertex.attrs.toArrows = arrow.attrs.fromVertex.attrs.toArrows.filter(item => (item.attrs.fromVertex!==arrow.attrs.fromVertex || item.attrs.toVertex!==arrow.attrs.toVertex))
      arrow.attrs.toVertex.attrs.fromArrows = arrow.attrs.toVertex.attrs.fromArrows.filter(item => (item.attrs.fromVertex!==arrow.attrs.fromVertex || item.attrs.toVertex!==arrow.attrs.toVertex))
      arrow.destroy()
      this.layer.draw()
    },

    deleteMachine(machine) {
      for (let m of machine.attrs.fromArrows) {
        this.deleteArrow(m)
      }
      for (let m of machine.attrs.toArrows) {
        this.deleteArrow(m)
      }
      machine.attrs.text.destroy()
      machine.destroy()
      this.layer.draw()
    },

    deleteQueue(queue) {
      for (let q of queue.attrs.fromArrows) {
        this.deleteArrow(q)
      }
      for (let q of queue.attrs.toArrows) {
        this.deleteArrow(q)
      }
      queue.attrs.text.destroy()
      queue.destroy()
      this.layer.draw()
    },

    endSimulation() {

      fetch('http://localhost:8080/end', {
        method: 'POST'
      })
      .catch((e)=> console.log(e))

      for (let text of this.stage.find('Text')) {
        if(text.id().includes('Q')) {
          text.text(text.text().split('\n')[0])
        }
      }
      for (let m of this.stage.find('Circle')) {
        m.fill('green')
      }
      this.connectState = 0
      this.selectedShapes = []
      this.selectedShape = false
      this.errorMessage = ''
      this.isRunning = false
    },

    resetGraph() {
      this.layer.destroyChildren()
      this.endSimulation()
      this.qCounter = 0
      this.mCounter = 0
    },

    replay() {
      fetch(`http://localhost:8080/replay`)
      .then(res=> res.json())
      .then((queue)=>{
        let updateObj = queue[0]
        if (updateObj.mid==='end') {
          this.isRunning = false
          for (let text of this.stage.find('Text')) {
            if(text.id().includes('Q')) {
              text.text(text.text().split('\n')[0])
            }
          }
          for (let m of this.stage.find('Circle')) {
            m.fill('green')
          }
          this.connectState = 0
          this.selectedShapes = []
          this.selectedShape = false
          this.errorMessage = ''
          return
        }
        updateObj.color==null? this.stage.find('#'+updateObj.mid)[0].fill('green') : this.stage.find('#'+updateObj.mid)[0].fill(updateObj.color)
        this.stage.find('#T'+updateObj.qid)[0].text(updateObj.qid+'\n'+updateObj.size)
        for (let i=0; i<queue.length-1; i++) {
          setTimeout(() => {
            updateObj = queue[i+1]
            if (updateObj.mid==='end') {
              this.isRunning = false
              for (let text of this.stage.find('Text')) {
                if(text.id().includes('Q')) {
                  text.text(text.text().split('\n')[0])
                }
              }
              for (let m of this.stage.find('Circle')) {
                m.fill('green')
              }
              this.connectState = 0
              this.selectedShapes = []
              this.selectedShape = false
              this.errorMessage = ''
              return
            }
            updateObj.color==null? this.stage.find('#'+updateObj.mid)[0].fill('green') : this.stage.find('#'+updateObj.mid)[0].fill(updateObj.color)
            this.stage.find('#T'+updateObj.qid)[0].text(updateObj.qid+'\n'+updateObj.size)
          }, queue[i+1].time-queue[0].time);
      }

      })
      .catch((e)=> console.log(e))
    }
  },
};
</script>

<style>
/* // made full app  take all  */

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #363f4f;
}

.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
/* // css for tool bar at the top of page */
.toolbar {
  width: 60%;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-top: 30px;

  /* margin-bottom: 20px; */
}

.btn {
  background-color: #3498db; /* Button background color */
  color: #ffffff; /* Text color */
  padding: 8px 16px; /* Padding inside the button */
  border: none; /* Remove button border */
  border-radius: 4px; /* Add some border-radius for rounded corners */
  cursor: pointer;
  transition: background-color 0.3s ease; /* Smooth transition for background color */
}

.btn:hover {
  background-color: #2980b9; /* Darker background color on hover */
}
/* // css for draw contain stage and layer */

.draw {
  margin-top: 30px;
  width: 90%;
  height: 80vh;
  /* position: relative; */
  /* left: 60px; */
  /* margin-left: 2vw; */
  /* margin-left: 200px; */
  /* margin-right: 8vw; */
  border: 1px solid black;
  border-radius: 5px;
  background-color: white;
  box-shadow: 5px 10px 10px 5px grey;
  /* overflow: scroll; */
}
</style>
