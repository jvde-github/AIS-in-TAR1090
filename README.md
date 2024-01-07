# AIS-in-tar1090

Through some creative manipulation, it is feasible to incorporate ship positions gathered by [AIS-catcher](https://github.com/jvde-github/AIS-catcher) into [tar1090](https://github.com/wiedehopf/tar1090), presenting both ships and planes in a unified display. However, it's important to note that this process can be somewhat cumbersome and should be undertaken cautiously, with appropriate backups in place. It's worth emphasizing that this is primarily a proof of concept to demonstrate visualizing AIS-catcher's GeoJSON outpyut, rather than a solution to include AIS data in tar1090.

## 1. AIS-catcher Configuration:
AIS-catcher provides the capability to produce GeoJSON data on its built-in webserver. Start AIS-catcher with the following command:
```console
AIS-catcher -N 8100 geojson on
```
Assuming your AIS-catcher server is running at 192.168.1.113, the data will be accessible at http://192.168.1.113:8100/geojson. This GeoJSON data can be effectively visualized using libraries like OpenLayers. Check out the `index.html` file in the codebase for a simple reference. To make it work with your specific setup, edit the line defining `server_aiscatcher`.

## 2. Integrating AIS Data with TAR1090:

Navigate to the directory that houses the tar1090 website files, typically located at /usr/local/share/tar1090/html.
    Locate the JavaScript file that starts with layers and has a .js extension (e.g., layersxxxxxxx.js).
    At the end of this JavaScript file, right before the `return layers_group;` statement, insert the following code snippet. 
```js
        const aiscatcher_server = "http://192.168.1.113:8100"; // update with your server address

        const aiscatcher_source = new ol.source.Vector({
            url: aiscatcher_server + '/geojson',
            format: new ol.format.GeoJSON()
        });

        function updateAIScatcher() { aiscatcher_source.refresh(); }
        setInterval(updateAIScatcher, 60000);

        const aiscatcher_mapping = {
            0: { size: [20, 20], offset: [120, 20], comment: 'CLASS_OTHER' },
            1: { size: [20, 20], offset: [120, 20], comment: 'CLASS_UNKNOWN' },
            2: { size: [20, 20], offset: [0, 20], comment: 'CLASS_CARGO' },
            3: { size: [20, 20], offset: [20, 20], comment: 'CLASS_B' },
            4: { size: [20, 20], offset: [40, 20], comment: 'CLASS_PASSENGER' },
            5: { size: [20, 20], offset: [60, 20], comment: 'CLASS_SPECIAL' },
            6: { size: [20, 20], offset: [80, 20], comment: 'CLASS_TANKER' },
            7: { size: [20, 20], offset: [100, 20], comment: 'CLASS_HIGHSPEED' },
            8: { size: [20, 20], offset: [140, 20], comment: 'CLASS_FISHING' },
            9: { size: [25, 25], offset: [0, 60], comment: 'CLASS_PLANE' },
            10: { size: [25, 25], offset: [0, 85], comment: 'CLASS_HELICOPTER' },
            11: { size: [20, 20], offset: [20, 40], comment: 'CLASS_STATION' },
            12: { size: [20, 20], offset: [0, 40], comment: 'CLASS_ATON' },
            13: { size: [20, 20], offset: [40, 40], comment: 'CLASS_SARTEPIRB' }
        };

        var aiscatcher_overlay = new ol.layer.Vector({
            type: 'overlay',
            title: "aiscatcher",
            name: "aiscatcher",
            zIndex: 99,
            source: aiscatcher_source,

            style: function (feature) {
                const cog = feature.get('cog');
                const rotation = (cog || 0) * (Math.PI / 180);
                const shipclass = feature.get('shipclass');
                const speed = feature.get('speed');

                const ofs = aiscatcher_mapping[shipclass].offset;
                const size = aiscatcher_mapping[shipclass].size;

                let o
                if (speed && speed > 0.5)
                    o = [ofs[0], 0];
                else
                    o = ofs;

                return new ol.style.Style({
                    image: new ol.style.Icon({
                        src: aiscatcher_server + '/icons.png',
                        anchor: [0.5, 0.5],
                        rotation: rotation,
                        size: size,
                        offset: o
                    })
                });
            }
        });
        world.push(aiscatcher_overlay);
```

This code will add an additional styled layer in the world group. **Note**: make sure you refresh the javascript files which are cached by default (I do that by opening layersxxxx.js in the browser and refreshing until I see the file with the code change). 

And then you have ships in tar1090:
![Screenshot from 2024-01-06 22-34-53](https://github.com/jvde-github/AIS-in-TAR1090/assets/52420030/2b715abf-64f5-4cc6-8f4b-30e8661e077c)

## 3. SDR-Enthusiasts Docker

For Docker, open a terminal in the docker 
```console
sudo docker exec -it tar1090 /bin/bash
```
and update `layersxxxxx.js` in `/usr/local/share/tar1090/html-webroot`. This step might have to be repeated every
time the container is restarted.

