<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
    <title>pydeck</title>
    <script src="https://api.tiles.mapbox.com/mapbox-gl-js/v1.13.0/mapbox-gl.js"></script>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css" />
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css" />
    <script src='https://cdn.jsdelivr.net/npm/@deck.gl/jupyter-widget@~8.8.*/dist/index.js'></script>
    <style>
      body {
        margin: 0;
        padding: 0;
        overflow: hidden;
      }

      #deck-container {
        width: 100vw;
        height: 100vh;
      }

      #deck-container canvas {
        z-index: 1;
        background: none;
      }
    </style>
  </head>
  <body>
    <div id="deck-container"></div>

    <script>
      const container = document.getElementById('deck-container');

      // 初始化视图设置
      const jsonInput = {
        "initialViewState": {
          "latitude": 32.44222723626717,
          "longitude": -97.08523170595782,
          "pitch": 50,
          "zoom": 14
        },
        "layers": [
          {
            "@@type": "PathLayer",
            "autoHighlight": true,
            "data": [],  // 初始为空，稍后动态更新
            "getColor": "@@=color",
            "getPath": "@@=path",
            "getWidth": 5,
            "dashJustified": true,  // 使用虚线来模拟箭头效果
            "dashArray": [5, 10],  // 设置虚线，5px 实线，10px 间隔
            "id": "dynamic-path-layer-1",
            "pickable": true,
            "widthMinPixels": 5,
            "widthScale": 10
          },
          {
            "@@type": "PathLayer",
            "autoHighlight": true,
            "data": [],  // 初始为空，稍后动态更新
            "getColor": "@@=color",
            "getPath": "@@=path",
            "getWidth": 5,
            "dashJustified": true,  // 使用虚线来模拟箭头效果
            "dashArray": [5, 10],  // 设置虚线
            "id": "dynamic-path-layer-2",
            "pickable": true,
            "widthMinPixels": 5,
            "widthScale": 10
          },
          {
            "@@type": "ScatterplotLayer",
            "data": [],
            "getPosition": "@@=position",
            "getColor": "@@=color",
            "getRadius": 100,
            "id": "collision-point-layer",
            "pickable": true
          }
        ],
        "mapProvider": "carto",
        "mapStyle": "https://basemaps.cartocdn.com/gl/dark-matter-gl-style/style.json",
        "views": [
          {
            "@@type": "MapView",
            "controller": true
          }
        ]
      };

      const tooltip = {'text': '{vehicle}'};
      const customLibraries = null;
      const configuration = null;

      // 创建deck.gl实例
      const deckInstance = createDeck({
        container,
        jsonInput,
        tooltip,
        customLibraries,
        configuration
      });

      // 轨迹数据从SHP文件生成
      const data1 = [{"path": [[-97.086584, 32.45046], [-97.085955, 32.450659], [-97.085215, 32.449324], [-97.084048, 32.444669], [-97.085729, 32.444016], [-97.085161, 32.442043], [-97.076269, 32.445307]], "color": [57, 88, 136, 204], "vehicle": "Vehicle 1"}];
      const data2 = [{"path": [[-97.07628, 32.445596], [-97.079027, 32.44458], [-97.080833, 32.443907], [-97.082522, 32.443256], [-97.083339, 32.44295], [-97.084141, 32.442647], [-97.084935, 32.442344], [-97.085724, 32.44204], [-97.08651, 32.441741], [-97.087295, 32.441445], [-97.088851, 32.440865], [-97.089629, 32.440577], [-97.091186, 32.439996], [-97.095958, 32.438227], [-97.096733, 32.437941], [-97.097506, 32.437664], [-97.098291, 32.4374], [-97.099092, 32.437155], [-97.099901, 32.436924], [-97.10072, 32.436712], [-97.101527, 32.436522], [-97.102306, 32.436362]], "color": [245, 219, 129, 204], "vehicle": "Vehicle 2"}];
      const collisionData = [{"position": [-97.08523170595782, 32.44222723626717], "color": [158, 0, 21], "vehicle": "Potential Collision"}];

      // 用于逐步显示两个轨迹点的函数，并在结束后重新开始
      async function updatePathLayer() {
        const maxLength = Math.max(data1[0].path.length, data2[0].path.length);  // 取两条轨迹的最大长度

        // 碰撞点的闪烁频率
        const collisionBlinkRate = 500; // 碰撞点每2秒闪烁一次
        let collisionBlink = true;

        while (true) {  // 无限循环，轨迹结束后重新开始
          for (let i = 1; i <= maxLength; i++) {
            // 动态更新两个车辆的路径，路径不会因为闪烁逻辑消失
            const updatedData1 = data1.map(d => ({
              path: d.path.slice(0, Math.min(i, d.path.length)),  // 逐步增加路径点
              color: d.color,
              vehicle: d.vehicle
            }));
            const updatedData2 = data2.map(d => ({
              path: d.path.slice(0, Math.min(i, d.path.length)),  // 逐步增加路径点
              color: d.color,
              vehicle: d.vehicle
            }));

            // 碰撞点的闪烁逻辑
            const showCollision = collisionBlink ? collisionData : [];

            // 更新路径层和碰撞点层
            const layers = [
              new deck.PathLayer({
                id: 'dynamic-path-layer-1',
                data: updatedData1,
                getPath: d => d.path,
                getColor: d => d.color,
                widthScale: 10,
                widthMinPixels: 5,
                getWidth: 5,
                dashJustified: true,
                dashArray: [5, 10],  // 使用虚线模拟箭头效果
                pickable: true,
                autoHighlight: true
              }),
              new deck.PathLayer({
                id: 'dynamic-path-layer-2',
                data: updatedData2,
                getPath: d => d.path,
                getColor: d => d.color,
                widthScale: 10,
                widthMinPixels: 5,
                getWidth: 5,
                dashJustified: true,
                dashArray: [5, 10],  // 使用虚线模拟箭头效果
                pickable: true,
                autoHighlight: true
              }),
              new deck.ScatterplotLayer({
                id: 'collision-point-layer',
                data: showCollision,
                getPosition: d => d.position,
                getColor: d => d.color,
                getRadius: 100,
                pickable: true
              })
            ];

            // 更新deck.gl实例的图层
            deckInstance.setProps({ layers });

            // 每次更新之间等待1秒
            await new Promise(resolve => setTimeout(resolve, 1000));

            // 切换碰撞点闪烁状态
            if (i % (collisionBlinkRate / 1000) === 0) {
              collisionBlink = !collisionBlink;
            }
          }
        }
      }

      // 启动轨迹更新
      updatePathLayer();

    </script>
  </body>
</html>
