<template>
  <div class="modeler">
    <div class="canvas" ref="canvas"></div>
    <div id="properties" ref="properties"></div>
  </div>
</template>

<script>
import BpmnModeler from "bpmn-js/lib/Modeler";
import "bpmn-js/dist/assets/diagram-js.css"; // 左边工具栏以及编辑节点的样式
import "bpmn-js/dist/assets/bpmn-font/css/bpmn.css";
import "bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css";
import "bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css";

import "bpmn-js-properties-panel/dist/assets/element-templates.css";
import "bpmn-js-properties-panel/dist/assets/properties-panel.css";
import camundaModdleDescriptors from "camunda-bpmn-moddle/resources/camunda";
import {
  BpmnPropertiesPanelModule,
  BpmnPropertiesProviderModule,
  CamundaPlatformPropertiesProviderModule,
} from "bpmn-js-properties-panel";
// use Camunda BPMN Moddle extension
// import CamundaExtensionModule from "camunda-bpmn-moddle/lib";
import customPalette from "./components/palette/index";
import paletteEntries from "./components/config/paletteEntries";
export default {
  data() {
    return {
      // bpmn建模器
      bpmnModeler: null,
      container: null,
      canvas: null,
      xmlStr: `<?xml version="1.0" encoding="UTF-8"?>
        <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" id="sid-38422fae-e03e-43a3-bef4-bd33b32041b2" targetNamespace="http://bpmn.io/bpmn" exporter="bpmn-js (https://demo.bpmn.io)" exporterVersion="6.0.4">
          <process id="Process_1" isExecutable="true">
           
          </process>
          <bpmndi:BPMNDiagram id="BpmnDiagram_1">
            <bpmndi:BPMNPlane id="BpmnPlane_1" bpmnElement="Process_1">
              
            </bpmndi:BPMNPlane>
          </bpmndi:BPMNDiagram>
        </definitions>`,
      processName: "",
      selectPanel: null,
      currentAttr: null,
      currentElement: {},
    };
  },
  mounted() {
    // 获取到属性ref为“content”的dom节点
    this.canvas = this.$refs.canvas;
    // 获取到属性ref为“canvas”的dom节点
    this.bpmnModeler = new BpmnModeler({
      propertiesPanel: {
        parent: "#properties",
      },
      container: this.canvas,
      paletteEntries,
      additionalModules: [
        customPalette,
        BpmnPropertiesPanelModule,
        BpmnPropertiesProviderModule,
        CamundaPlatformPropertiesProviderModule,
        // CamundaExtensionModule,
      ],
      moddleExtensions: {
        camunda: camundaModdleDescriptors,
      },
    });
    this.addEventBusListerner();
    this.createNewDiagram();
  },
  created() {},

  methods: {
    getShapeById(id) {
      const elementRegistry = this.bpmnModeler.get("elementRegistry");
      return elementRegistry.get(id);
    },
    async createNewDiagram() {
      const bpmnXmlStr = `<?xml version="1.0" encoding="UTF-8"?>
        <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" id="sid-38422fae-e03e-43a3-bef4-bd33b32041b2" targetNamespace="http://bpmn.io/bpmn" exporter="bpmn-js (https://demo.bpmn.io)" exporterVersion="6.0.4">
          <process id="Process_1" isExecutable="true">
           
          </process>
          <bpmndi:BPMNDiagram id="BpmnDiagram_1">
            <bpmndi:BPMNPlane id="BpmnPlane_1" bpmnElement="Process_1">
              <bpmndi:BPMNShape id="StartEvent_1acpsqt_di" bpmnElement="StartEvent_1acpsqt">
                <omgdc:Bounds x="152" y="82" width="36" height="36" />
              </bpmndi:BPMNShape>
            </bpmndi:BPMNPlane>
          </bpmndi:BPMNDiagram>
        </definitions>`;
      // 将字符串转换成图显示出来
      try {
        const { warings } = await this.bpmnModeler.importXML(bpmnXmlStr);
        // addEventBusListerner();
        // 调整在正中间
        this.bpmnModeler.get("canvas").zoom("fit-viewport", "auto");
        // console.log('rendered')
      } catch (err) {
        // console.log('error rendering', err)
      }
      console.log(this.bpmnModeler);
    },
    addEventBusListerner() {
      let id = "";
      const eventBus = this.bpmnModeler.get("eventBus");
      //单独给进程设置样式
      console.log(eventBus);

      // 注册节点事件，eventTypes中可以写多个事件
      const eventTypes = [
        "element.click",
        "element.changed",
        "element.dblclick",
      ];
      eventTypes.forEach((eventType) => {
        eventBus.on(eventType, (e) => {
          const { element } = e;
          console.log(element);
          if (eventType == "element.changed") {
            let element = e.element;
            console.log(element);
            if (element.type == "bpmn:UserTask") {
              console.log(this.selectPanel.$el);
              // selectPanel.value.$el.style.display = 'block'
              this.isshow = true;
            } else {
              // selectPanel.value.$el.style.display = 'none'
              this.isshow = false;
            }
          }

          if (eventType === "element.click") {
            console.log(element);

            if (element.type !== "bpmn:UserTask") {
              this.isshow = false;
            } else {
              this.isshow = true;
            }

            // 节点点击后想要做的处理
            // 此时想要点击节点后，拿到节点实例，通过外部输入更新节点名称
            this.currentElement = element;

            console.log(this.currentElement);
            //传递数据
            // this.selectPanel.currentElement = this.currentElement;
            console.log(this.currentElement.id);
            id = this.currentElement.id;
            const ele = this.getShapeById(id);
            if (element.type === "bpmn:UserTask") {
              this.isshow = true;
            }
            console.log(this.currentElement.businessObject);
            this.currentAttr = this.currentElement.businessObject;
          }
          if (eventType == "element.dblclick") {
            this.isshow = false;
          }
        });
      });
    },
  },
};
</script>

<style scoped lang="less">
.modeler {
  height: 100%;
  display: flex;
  justify-content: space-between;
  .canvas {
    width: 90%;
    // height: 100%;
  }
}
</style>
