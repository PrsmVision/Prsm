// quartz/components/AsciiHeader.tsx

import { QuartzComponent, QuartzComponentConstructor } from "./types"

interface Options {
  text: string
  height?: number
  asciiFontSize?: number
  textFontSize?: number
  textColor?: string
  enableWaves?: boolean
}

export default ((opts?: Partial<Options>) => {
  const text          = opts?.text          ?? "PrismaVision"
  const height        = opts?.height        ?? 180
  const asciiFontSize = opts?.asciiFontSize ?? 8
  const textFontSize  = opts?.textFontSize  ?? 200
  const textColor     = opts?.textColor     ?? "#fdf9f3"
  const enableWaves   = opts?.enableWaves   ?? true

  const AsciiHeader: QuartzComponent = () => {
    return (
      <div
        id="ascii-header-root"
        style={`height:${height}px; width:100%; position:relative; overflow:hidden;`}
        data-text={text}
        data-ascii-font-size={String(asciiFontSize)}
        data-text-font-size={String(textFontSize)}
        data-text-color={textColor}
        data-enable-waves={String(enableWaves)}
      />
    )
  }

  AsciiHeader.css = `
#ascii-header-root {
  display: block;
  background: transparent;
  pointer-events: auto;
}
#ascii-header-root pre {
  margin: 0 !important;
  padding: 0 !important;
  user-select: none;
  line-height: 1em !important;
  position: absolute;
  left: 0;
  top: 0;
  background-image: radial-gradient(circle, #ff6188 0%, #fc9867 50%, #ffd866 100%);
  background-attachment: fixed;
  -webkit-text-fill-color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  z-index: 9;
  mix-blend-mode: difference;
}
#ascii-header-root canvas {
  position: absolute;
  left: 0; top: 0;
  width: 100% !important;
  height: 100% !important;
  image-rendering: pixelated;
}
`

  AsciiHeader.afterDOMLoaded = `
(function () {
  // ── Roda só uma vez por carregamento de página ─────────────────
  // O afterDOMLoaded é re-executado a cada nav no Quartz SPA,
  // então guardamos tudo no window para não duplicar.
  if (window.__ascii_registered) return;
  window.__ascii_registered = true;

  const PX = window.devicePixelRatio || 1;
  Math.map = (n,a,b,c,d) => ((n-a)/(b-a))*(d-c)+c;

  const vert = \`
    varying vec2 vUv;
    uniform float uTime;
    uniform float uEnableWaves;
    void main(){
      vUv=uv;
      float t=uTime*5.;float w=uEnableWaves;
      vec3 p=position;
      p.x+=sin(t+position.y)*.5*w;
      p.y+=cos(t+position.z)*.15*w;
      p.z+=sin(t+position.x)*w;
      gl_Position=projectionMatrix*modelViewMatrix*vec4(p,1.0);
    }
  \`;
  const frag = \`
    varying vec2 vUv;
    uniform float uTime;
    uniform sampler2D uTexture;
    void main(){
      float t=uTime;vec2 pos=vUv;
      float r=texture2D(uTexture,pos+cos(t*2.-t+pos.x)*.01).r;
      float g=texture2D(uTexture,pos+tan(t*.5+pos.x-t)*.01).g;
      float b=texture2D(uTexture,pos-cos(t*2.+t+pos.y)*.01).b;
      float a=texture2D(uTexture,pos).a;
      gl_FragColor=vec4(r,g,b,a);
    }
  \`;

  class AsciiFilter {
    constructor(renderer,{fontSize,fontFamily,charset,invert}={}){
      this.renderer=renderer;
      this.domElement=document.createElement("div");
      this.domElement.style.cssText="position:absolute;top:0;left:0;width:100%;height:100%;";
      this.pre=document.createElement("pre");
      this.domElement.appendChild(this.pre);
      this.canvas=document.createElement("canvas");
      this.context=this.canvas.getContext("2d");
      this.domElement.appendChild(this.canvas);
      this.deg=0;
      this.invert=invert??true;
      this.fontSize=fontSize??12;
      this.fontFamily=fontFamily??"'Courier New',monospace";
      this.charset=charset??" .'^\\",:;Il!i~+_-?][}{1)(|/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$";
      const c=this.context;
      c.webkitImageSmoothingEnabled=c.mozImageSmoothingEnabled=c.msImageSmoothingEnabled=c.imageSmoothingEnabled=false;
      this._mv=this._mv.bind(this);
      document.addEventListener("mousemove",this._mv);
    }
    setSize(w,h){
      this.width=w;this.height=h;
      this.renderer.setSize(w,h);
      this._reset();
      this.center={x:w/2,y:h/2};
      this.mouse={x:w/2,y:h/2};
    }
    _reset(){
      const c=this.context;
      c.font=\`\${this.fontSize}px \${this.fontFamily}\`;
      const cw=c.measureText("A").width;
      this.cols=Math.floor(this.width/(this.fontSize*(cw/this.fontSize)));
      this.rows=Math.floor(this.height/this.fontSize);
      this.canvas.width=this.cols;this.canvas.height=this.rows;
      Object.assign(this.pre.style,{fontFamily:this.fontFamily,fontSize:this.fontSize+"px",
        margin:"0",padding:"0",lineHeight:"1em",position:"absolute",left:"0",top:"0",
        zIndex:"9",backgroundAttachment:"fixed",mixBlendMode:"difference"});
    }
    render(scene,camera){
      this.renderer.render(scene,camera);
      const {width:w,height:h}=this.canvas;
      this.context.clearRect(0,0,w,h);
      if(w&&h)this.context.drawImage(this.renderer.domElement,0,0,w,h);
      this._ascii(w,h);this._hue();
    }
    _mv(e){this.mouse={x:e.clientX*PX,y:e.clientY*PX};}
    get dx(){return(this.mouse?.x??0)-this.center.x;}
    get dy(){return(this.mouse?.y??0)-this.center.y;}
    _hue(){
      const d=(Math.atan2(this.dy,this.dx)*180)/Math.PI;
      this.deg+=(d-this.deg)*0.075;
      this.domElement.style.filter=\`hue-rotate(\${this.deg.toFixed(1)}deg)\`;
    }
    _ascii(w,h){
      if(!w||!h)return;
      const data=this.context.getImageData(0,0,w,h).data;
      let str="";
      for(let y=0;y<h;y++){
        for(let x=0;x<w;x++){
          const i=(x+y*w)*4;
          const[r,g,b,a]=[data[i],data[i+1],data[i+2],data[i+3]];
          if(a===0){str+=" ";continue;}
          let gray=(0.3*r+0.6*g+0.1*b)/255;
          let idx=Math.floor((1-gray)*(this.charset.length-1));
          if(this.invert)idx=this.charset.length-idx-1;
          str+=this.charset[idx];
        }
        str+="\\n";
      }
      this.pre.innerHTML=str;
    }
    dispose(){document.removeEventListener("mousemove",this._mv);}
  }

  class CanvasTxt {
    constructor(txt,{fontSize=200,fontFamily="Arial",color="#fdf9f3"}={}){
      this.canvas=document.createElement("canvas");
      this.ctx=this.canvas.getContext("2d");
      this.txt=txt;this.font=\`600 \${fontSize}px \${fontFamily}\`;this.color=color;
      this.ctx.font=this.font;
      const m=this.ctx.measureText(txt);
      this.canvas.width=Math.ceil(m.width)+20;
      this.canvas.height=Math.ceil(m.actualBoundingBoxAscent+m.actualBoundingBoxDescent)+20;
    }
    render(){
      const c=this.ctx;c.clearRect(0,0,this.canvas.width,this.canvas.height);
      c.fillStyle=this.color;c.font=this.font;
      const m=c.measureText(this.txt);
      c.fillText(this.txt,10,10+m.actualBoundingBoxAscent);
    }
    get width(){return this.canvas.width;}
    get height(){return this.canvas.height;}
    get texture(){return this.canvas;}
  }

  class CanvAscii {
    constructor({text,asciiFontSize,textFontSize,textColor,planeBaseHeight,enableWaves},container,w,h){
      this.container=container;this.width=w;this.height=h;
      this.textString=text;this.asciiFontSize=asciiFontSize;this.textFontSize=textFontSize;
      this.textColor=textColor;this.planeBaseHeight=planeBaseHeight;this.enableWaves=enableWaves;
      this.mouse={x:w/2,y:h/2};
      this.camera=new THREE.PerspectiveCamera(45,w/h,1,1000);
      this.camera.position.z=30;
      this.scene=new THREE.Scene();
      this._mv=this._mv.bind(this);
    }
    async init(){
      try{await document.fonts.load(\`600 200px "IBM Plex Mono"\`);}catch(_){}
      await document.fonts.ready;
      this._mesh();this._renderer();
    }
    _mesh(){
      this.tc=new CanvasTxt(this.textString,{fontSize:this.textFontSize,fontFamily:"IBM Plex Mono,monospace",color:this.textColor});
      this.tc.render();
      this.tex=new THREE.CanvasTexture(this.tc.texture);
      this.tex.minFilter=THREE.NearestFilter;
      const asp=this.tc.width/this.tc.height,bh=this.planeBaseHeight;
      this.geo=new THREE.PlaneGeometry(bh*asp,bh,36,36);
      this.mat=new THREE.ShaderMaterial({vertexShader:vert,fragmentShader:frag,transparent:true,
        uniforms:{uTime:{value:0},mouse:{value:1},uTexture:{value:this.tex},uEnableWaves:{value:this.enableWaves?1:0}}});
      this.mesh=new THREE.Mesh(this.geo,this.mat);
      this.scene.add(this.mesh);
    }
    _renderer(){
      this.rdr=new THREE.WebGLRenderer({antialias:false,alpha:true});
      this.rdr.setPixelRatio(1);this.rdr.setClearColor(0x000000,0);
      this.flt=new AsciiFilter(this.rdr,{fontFamily:"IBM Plex Mono,monospace",fontSize:this.asciiFontSize,invert:true});
      this.container.appendChild(this.flt.domElement);
      this.setSize(this.width,this.height);
      this.container.addEventListener("mousemove",this._mv);
      this.container.addEventListener("touchmove",this._mv);
    }
    setSize(w,h){
      this.width=w;this.height=h;
      this.camera.aspect=w/h;this.camera.updateProjectionMatrix();
      this.flt.setSize(w,h);this.center={x:w/2,y:h/2};
    }
    _mv(evt){
      const e=evt.touches?evt.touches[0]:evt;
      const b=this.container.getBoundingClientRect();
      this.mouse={x:e.clientX-b.left,y:e.clientY-b.top};
    }
    load(){
      const loop=()=>{this.raf=requestAnimationFrame(loop);this._render();};
      loop();
    }
    _render(){
      const t=Date.now()*.001;
      this.tc.render();this.tex.needsUpdate=true;
      this.mat.uniforms.uTime.value=Math.sin(t);
      const rx=Math.map(this.mouse.y,0,this.height,.5,-.5);
      const ry=Math.map(this.mouse.x,0,this.width,-.5,.5);
      this.mesh.rotation.x+=(rx-this.mesh.rotation.x)*.05;
      this.mesh.rotation.y+=(ry-this.mesh.rotation.y)*.05;
      this.flt.render(this.scene,this.camera);
    }
    dispose(){
      cancelAnimationFrame(this.raf);
      this.flt?.dispose();
      if(this.flt?.domElement?.parentNode)this.container.removeChild(this.flt.domElement);
      this.container.removeEventListener("mousemove",this._mv);
      this.container.removeEventListener("touchmove",this._mv);
      this.scene.traverse(o=>{
        if(o.isMesh){
          o.geometry?.dispose();
          if(o.material){Object.values(o.material).forEach(v=>v?.dispose?.());o.material.dispose();}
        }
      });
      this.scene.clear();
      this.rdr?.dispose();this.rdr?.forceContextLoss();
    }
  }

  // ── Estado global ── guardado no window para sobreviver a navs ──
  window.__ascii = window.__ascii || { instance: null, ro: null };

  async function initHeader() {
    const state = window.__ascii;

    // Limpa instância anterior
    if (state.ro)       { state.ro.disconnect(); state.ro = null; }
    if (state.instance) { state.instance.dispose(); state.instance = null; }

    const ROOT = document.getElementById("ascii-header-root");
    if (!ROOT) return;

    // Pequeno delay pra garantir que o DOM do Quartz terminou de montar
    await new Promise(r => setTimeout(r, 50));

    const { width, height } = ROOT.getBoundingClientRect();
    if (!width || !height) return;

    const TEXT   = ROOT.dataset.text        || "PrismaVision";
    const AFS    = Number(ROOT.dataset.asciiFontSize) || 8;
    const TFS    = Number(ROOT.dataset.textFontSize)  || 200;
    const COLOR  = ROOT.dataset.textColor   || "#fdf9f3";
    const WAVES  = ROOT.dataset.enableWaves !== "false";

    state.instance = new CanvAscii(
      { text:TEXT, asciiFontSize:AFS, textFontSize:TFS, textColor:COLOR, planeBaseHeight:8, enableWaves:WAVES },
      ROOT, width, height
    );
    await state.instance.init();
    state.instance.load();

    state.ro = new ResizeObserver(([e]) => {
      const {width:w,height:h} = e.contentRect;
      if(w&&h) state.instance?.setSize(w,h);
    });
    state.ro.observe(ROOT);
  }

  function start() {
    if (window.THREE) { initHeader(); return; }
    const s = document.createElement("script");
    s.src = "https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js";
    s.onload = initHeader;
    document.head.appendChild(s);
  }

  // Listener de nav registrado só uma vez
  document.addEventListener("nav", initHeader);
  start();
})();
`

  return AsciiHeader
}) satisfies QuartzComponentConstructor