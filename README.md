# easy-view

> 一个从各种意义上来讲都很简单的 MVP 框架，UI 和逻辑分离，支持运行在 worker 等 js 容器中

## 使用说明

```bash
npm i @easythings/easy-view
```

main.jsx

```jsx
import { mount, mountFromPort } from "@easythings/easy-view";

const worker = new Worker(new URL("./worker.jsx", import.meta.url));
const sharedWorker = new SharedWorker(
  new URL("./sharedWorker.jsx", import.meta.url)
);

function Main(props, children, eventHandlers, hook) {
  return "Hello World";
}

const container1 = document.createElement("div");
const container2 = document.createElement("div");
const container3 = document.createElement("div");
document.body.append(container1, container2, container3);

mount(<Main />, container1); // 主线程用法

mountFromPort(worker, container2); // worker用法

mountFromPort(sharedWorker.port, container3); // sharedWorker用法
```

worker.jsx/sharedWorker.jsx

```jsx
import { render } from "@easythings/easy-view";

const listen = render(<WorkerComponent hello="world" />);

init();

function init() {
  const platform = self.constructor.name;
  const handler = {
    ["SharedWorkerGlobalScope"]() {
      self.onconnect = ({ ports }) => {
        ports.forEach(port => listen(port));
      };
    },
    ["DedicatedWorkerGlobalScope"]() {
      listen(self);
    }
  };
  handler[platform]?.();
}

function WorkerComponent(props, children, eventHandlers, hook) {
  let i = 0;
  let j = 0;
  let interval = setInterval(() => {
    j++;
    hook.dispatch("update"); // 发现view没有按照预期更新可以显式调用更新
  }, 1000);

  hook.on("destroy", () => {
    interval = clearInterval(interval); // 卸载时候要注意清除掉闭包里的引用等等
  });

  const onClick = next => {
    i++;
    console.log("事件从上向下", next.event);
    next();
    console.log("再从下向上");
  };
  const onClick2 = next => {
    console.log("可以用next(false)阻止向下");
    next(false);
    console.log("但是从下往上没办法阻止");
  };
  const onClick3 = next => {
    console.log("123321");
  };

  return (
    <div on-click={onClick}>
      <div on-click={onClick2}>
        <div on-click={onClick3}>
          {i}-{j}
        </div>
      </div>
    </div>
  );
}
```

.babelrc 需要配置下面的预设和插件

```json
{
  "presets": [
    [
      "@babel/preset-react",
      {
        "runtime": "automatic",
        "importSource": "@easythings/easy-view"
      }
    ]
  ],
  "plugins": [["@easythings/easy-view-jsx"]]
}
```

## [CHANGELOG](./CHANGELOG.md)
