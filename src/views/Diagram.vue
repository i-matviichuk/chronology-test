<template>
  <!--  <div style="border: solid 1px black; width:900px; height:900px">-->
  <div id="sample">
    <div
      id="myDiagramDiv"
      style="background-color: #696969; height: 100vh"
    />
    <div>
      <div id="myInspector" />
    </div>
    <div>
      <div>
        <!--        <button id="SaveButton" onclick="save()">Save</button>-->
      <!--        <button @click="load()">-->
      <!--          Load-->
      <!--        </button>-->
      </div>
    </div>
  </div>
</template>


<script>
import go from 'gojs';
import diagramData from '../assets/treeData';

const $ = go.GraphObject.make;

export default {
  name: 'Diagram',
  data() {
    return {
      diagram: null,
      diagramData,
    };
  },
  mounted() {
    this.diagram = $(go.Diagram, 'myDiagramDiv', {
      maxSelectionCount: 2, // users can select only one part at a time
      validCycle: go.Diagram.CycleDestinationTree, // make sure users can only create trees

      // 'clickCreatingTool.archetypeNodeData': { // allow double-click in background to
      // create a new node
      //   name: '(new person)',
      //   title: '',
      //   comments: '',
      // },
      // eslint-disable-next-line func-names
      // 'clickCreatingTool.insertPart': function (loc) { // scroll to the new node
      //   const node = go.ClickCreatingTool.prototype.insertPart.call(this, loc);
      //   if (node !== null) {
      //     this.diagram.select(node);
      //     this.diagram.commandHandler.scrollToPart(node);
      //     this.diagram.commandHandler.editTextBlock(node.findObject('NAMETB'));
      //   }
      //   return node;
      // },
      layout:
          $(go.TreeLayout,
            {
              treeStyle: go.TreeLayout.StyleLastParents,
              arrangement: go.TreeLayout.ArrangementHorizontal,
              // properties for most of the tree:
              angle: 90,
              layerSpacing: 35,
              // properties for the "last parents":
              alternateAngle: 90,
              alternateLayerSpacing: 35,
              alternateAlignment: go.TreeLayout.AlignmentBus,
              alternateNodeSpacing: 20,
            }),
      // 'undoManager.isEnabled': true, // enable undo & redo
    });

    // this.diagram.addDiagramListener('Modified', () => {
    //   const button = document.getElementById('SaveButton');
    //   if (button) button.disabled = !this.diagram.isModified;
    //   const idx = document.title.indexOf('*');
    //   if (this.diagram.isModified) {
    //     if (idx < 0) document.title += '*';
    //   } else {
    //     document.title = document.title.substr(0, idx);
    //   }
    // });

    // manage boss info manually when a node or link is deleted from the diagram
    // this.diagram.addDiagramListener('SelectionDeleting', (e) => {
    //   const part = e.subject.first(); // e.subject is the diagram.selection collection,
    //   // so we'll get the first since we know we only have one selection
    //   this.diagram.startTransaction('clear boss');
    //   if (part instanceof go.Node) {
    //     const it = part.findTreeChildrenNodes(); // find all child nodes
    //     while (it.next()) { // now iterate through them and clear out the boss information
    //       const child = it.value;
    //       const bossText = child.findObject('boss'); // since the boss TextBlock is named,
    //       we can access it by name
    //       if (bossText === null) return;
    //       bossText.text = '';
    //     }
    //   } else if (part instanceof go.Link) {
    //     const child = part.toNode;
    //     const bossText = child.findObject('boss'); // since the boss TextBlock is named,
    //     we can access it by name
    //     if (bossText === null) return;
    //     bossText.text = '';
    //   }
    //   this.diagram.commitTransaction('clear boss');
    // });

    const levelColors = ['#AC193D', '#2672EC', '#8C0095', '#5133AB',
      '#008299', '#D24726', '#008A00', '#094AB2'];

    // override TreeLayout.commitNodes to also modify the
    // background brush based on the tree depth level
    // eslint-disable-next-line func-names
    this.diagram.layout.commitNodes = function () {
      go.TreeLayout.prototype.commitNodes.call(this.diagram.layout); // do the standard behavior
      // then go through all of the vertexes and set their corresponding node's Shape.fill
      // to a brush dependent on the TreeVertex.level value
      this.diagram.layout.network.vertexes.each((v) => {
        if (v.node) {
          const level = v.level % (levelColors.length);
          const color = levelColors[level];
          const shape = v.node.findObject('SHAPE');
          if (shape) {
            shape.fill = $(go.Brush, 'Linear', {
              0: color,
              1: go.Brush.lightenBy(color, 0.05),
              start: go.Spot.Left,
              end: go.Spot.Right,
            });
          }
        }
      });
    };

    function nodeDoubleClick(e, obj) {
      const clicked = obj.part;
      if (clicked !== null) {
        const thisemp = clicked.data;
        this.diagram.startTransaction('add employee');
        const newemp = {
          name: '(new person)',
          lifetime: '',
          comments: '',
          parent: thisemp.key,
        };
        this.diagram.model.addNodeData(newemp);
        this.diagram.commitTransaction('add employee');
      }
    }

    // this is used to determine feedback during drags
    // function mayWorkFor(node1, node2) {
    //   if (!(node1 instanceof go.Node)) return false; // must be a Node
    //   if (node1 === node2) return false; // cannot work for yourself
    //   if (node2.isInTreeOf(node1)) return false; // cannot work for someone who works for you
    //   return true;
    // }

    // This function provides a common style for most of the TextBlocks.
    // Some of these values may be overridden in a particular TextBlock.
    function textStyle() {
      return { font: '9pt  Segoe UI,sans-serif', stroke: 'white' };
    }

    // This converter is used by the Picture.
    // function findHeadShot(key) {
    //   if (key < 0 || key > 16) return 'images/HSnopic.png'; // There are only
    //   16 images on the server
    //   return `images/HS${key}.png`;
    // }

    // define the Node template
    this.diagram.nodeTemplate = $(go.Node, 'Auto',
      { doubleClick: nodeDoubleClick },
      // { // handle dragging a Node onto a Node to (maybe) change the reporting relationship
      //   // eslint-disable-next-line no-unused-vars
      //   mouseDragEnter(e, node, prev) {
      //     this.diagram = node.diagram; // ///////////////////////ERROR IS HERE Cannot set
      //     // property 'diagram' of undefined
      //     const selnode = this.diagram.selection.first();
      //     if (!mayWorkFor(selnode, node)) return;
      //     const shape = node.findObject('SHAPE');
      //     if (shape) {
      //       // eslint-disable-next-line no-underscore-dangle
      //       shape._prevFill = shape.fill; // remember the original brush
      //       shape.fill = 'darkred';
      //     }
      //   },
      //   // eslint-disable-next-line no-unused-vars
      //   mouseDragLeave(e, node, next) {
      //     const shape = node.findObject('SHAPE');
      //     // eslint-disable-next-line no-underscore-dangle
      //     if (shape && shape._prevFill) {
      //       // eslint-disable-next-line no-underscore-dangle
      //       shape.fill = shape._prevFill; // restore the original brush
      //     }
      //   },
      //   mouseDrop(e, node) {
      //     this.diagram = node.diagram;
      //     const selnode = this.diagram.selection.first(); // assume just one Node in selection
      //     if (mayWorkFor(selnode, node)) {
      //       // find any existing link into the selected node
      //       const link = selnode.findTreeParentLink();
      //       if (link !== null) { // reconnect any existing link
      //         link.fromNode = node;
      //       } else { // else create a new link
      //         this.diagram.toolManager.linkingTool
      //           .insertLink(node, node.port, selnode, selnode.port);
      //       }
      //     }
      //   },
      // },
      // for sorting, have the Node.text be the data.name
      new go.Binding('text', 'name'),
      // bind the Part.layerName to control the Node's layer depending on whether it isSelected
      new go.Binding('layerName', 'isSelected', (sel => (sel ? 'Foreground' : ''))).ofObject(),
      // define the node's outer shape
      $(go.Shape, 'Rectangle',
        {
          name: 'SHAPE',
          fill: 'white',
          stroke: null,
          // set the port properties:
          portId: '',
          fromLinkable: true,
          toLinkable: true,
          cursor: 'pointer',
        }),
      $(go.Panel, 'Horizontal',
        // $(go.Picture,
        //   {
        //     name: 'Picture',
        //     desiredSize: new go.Size(39, 50),
        //     margin: new go.Margin(6, 8, 6, 10),
        //   },
        //   new go.Binding('source', 'key', findHeadShot)),
        // define the panel where the text will appear
        $(go.Panel, 'Table',
          {
            maxSize: new go.Size(150, 999),
            margin: new go.Margin(6, 10, 0, 3),
            defaultAlignment: go.Spot.Left,
          },
          $(go.RowColumnDefinition, { column: 2, width: 4 }),
          $(go.TextBlock, textStyle(), // the name
            {
              row: 0,
              column: 0,
              columnSpan: 5,
              font: '12pt Segoe UI,sans-serif',
              editable: true,
              isMultiline: false,
              minSize: new go.Size(10, 16),
            },
            new go.Binding('text', 'name').makeTwoWay()),
          $(go.TextBlock, 'Title: ', textStyle(),
            { row: 1, column: 0 }),
          $(go.TextBlock, textStyle(),
            {
              row: 1,
              column: 1,
              columnSpan: 4,
              editable: true,
              isMultiline: false,
              minSize: new go.Size(10, 14),
              margin: new go.Margin(0, 0, 0, 3),
            },
            new go.Binding('text', 'title').makeTwoWay()),
          $(go.TextBlock, textStyle(),
            { row: 2, column: 0 },
            new go.Binding('text', 'key', (v => `ID: ${v}`))),
          $(go.TextBlock, textStyle(),
            { name: 'father', row: 3, column: 0 }, // we include a name so we can access this TextBlock when deleting Nodes/Links
            new go.Binding('text', 'parent', (v => `Father: ${v}`))),
          $(go.TextBlock, textStyle(), // the comments
            {
              row: 3,
              column: 0,
              columnSpan: 5,
              font: 'italic 9pt sans-serif',
              wrap: go.TextBlock.WrapFit,
              editable: true, // by default newlines are allowed
              minSize: new go.Size(10, 14),
            },
            new go.Binding('text', 'comments').makeTwoWay())), // end Table Panel
        // eslint-disable-next-line function-paren-newline
      ), // end Horizontal Panel
      // eslint-disable-next-line function-paren-newline
    ); // end Node

    // the context menu allows users to make a position vacant,
    // remove a role and reassign the subtree, or remove a department
    // this.diagram.nodeTemplate.contextMenu =
    //   $('ContextMenu',
    //     $('ContextMenuButton',
    //       $(go.TextBlock, 'Vacate Position'),
    //       {
    //         click(e, obj) {
    //           const node = obj.part.adornedPart;
    //           if (node !== null) {
    //             const thisemp = node.data;
    //             this.diagram.startTransaction('vacate');
    //             // update the key, name, and comments
    //             this.diagram.model.setDataProperty(thisemp, 'name', '(Vacant)');
    //             this.diagram.model.setDataProperty(thisemp, 'comments', '');
    //             this.diagram.commitTransaction('vacate');
    //           }
    //         },
    //       },
    //     ),
    //     $('ContextMenuButton',
    //       $(go.TextBlock, 'Remove Role'),
    //       {
    //         click(e, obj) {
    //           // reparent the subtree to this node's boss, then remove the node
    //           const node = obj.part.adornedPart;
    //           if (node !== null) {
    //             this.diagram.startTransaction('reparent remove');
    //             const chl = node.findTreeChildrenNodes();
    //             // iterate through the children and set
    //             // their parent key to our selected node's parent key
    //             while (chl.next()) {
    //               const emp = chl.value;
    //               this.diagram.model.setParentKeyForNodeData(emp.data,
    //                 node.findTreeParentNode().data.key);
    //             }
    //             // and now remove the selected node itself
    //             this.diagram.model.removeNodeData(node.data);
    //             this.diagram.commitTransaction('reparent remove');
    //           }
    //         },
    //       },
    //     ),
    //     $('ContextMenuButton',
    //       $(go.TextBlock, 'Remove Department'),
    //       {
    //         click(e, obj) {
    //           // remove the whole subtree, including the node itself
    //           const node = obj.part.adornedPart;
    //           if (node !== null) {
    //             this.diagram.startTransaction('remove dept');
    //             this.diagram.removeParts(node.findTreeParts());
    //             this.diagram.commitTransaction('remove dept');
    //           }
    //         },
    //       },
    //     ),
    //   );

    // define the Link template
    this.diagram.linkTemplate = $(go.Link, go.Link.Orthogonal,
      { corner: 5, relinkableFrom: true, relinkableTo: true },
      $(go.Shape, { strokeWidth: 4, stroke: '#00a4a4' })); // the link shape

    // read in the JSON-format data from the "mySavedModel" element
    this.load();


    // support editing the properties of the selected person in HTML
    // if (window.Inspector) {
    //   myInspector = new Inspector('myInspector', diagram,
    //     {
    //       properties: {
    //         key: { readOnly: true },
    //         comments: {},
    //       },
    //     });
    // }
  },
  methods: {

    // Show the diagram's model in JSON format
    // save() {
    //   document.getElementById('mySavedModel').value = diagram.model.toJson();
    //   diagram.isModified = false;
    // },
    //
    load() {
      this.diagram.model = go.Model.fromJson(this.diagramData);
      // make sure new data keys are unique positive integers
      let lastkey = 1;
      // eslint-disable-next-line func-names
      this.diagram.model.makeUniqueKeyFunction = function (model, data) {
        let k = data.key || lastkey;
        // eslint-disable-next-line no-plusplus
        while (model.findNodeDataForKey(k)) k++;
        lastkey = data.key;
        k = lastkey;

        return k;
      };
    },
  },
};

// export default {
//   name: 'Diagram',
//   mounted() {
//     const diagram =
//       $(go.Diagram, this.$el, {
//         'undoManager.isEnabled': true,
//         'grid.visible': true,
//         'grid.gridCellSize': new go.Size(30, 30),
//         // draggingTool: new GuidedDraggingTool(),loader
//       });
//
//     diagram.nodeTemplate =
//       $(go.Node, 'Vertical',
//         {
//           locationSpot: go.Spot.Center,
//           locationObjectName: 'SHAPE',
//           selectionAdorned: false,
//           resizable: true,
//           resizeObjectName: 'SHAPE',
//           rotatable: true,
//           rotateObjectName: 'SHAPE',
//           // eslint-disable-next-line no-bitwise
//           layoutConditions: go.Part.LayoutStandard & ~go.Part.LayoutNodeSized,
//         },
//         new go.Binding('layerName', 'isHighlighted', (h => (h ? 'Foreground' : ''))).ofObject(),
//         $(go.Shape,
//           {
//             name: 'SHAPE',
//             width: 70,
//             height: 70,
//             stroke: '#C2185B',
//             fill: '#F48FB1',
//             strokeWidth: 3,
//           },
//         ),
//       );
//     diagram.startTransaction('new object');
//     diagram.model.addNodeData({});
//     diagram.model.addNodeData({});
//     diagram.commitTransaction('new object');
//   },
// };
</script>
