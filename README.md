# AIS-in-TAR1090

AIS-catcher has the ability to produce GeoJSON data on its built-in webserver, for example run with:
```console
AIS-catcher -N 8100 geojson on
```
If the server runs at `192.168.1.113`, the data will be available at `http://192.168.1.113:8100/geojson`. This data can easily be plotted using a library like openlayers. See `index.html` inn the codebase. To make this run on your setup, edit the line that defines `server_aiscatcher`.

With some fiddling you can include this layer in TAR1090 as well. Go to the directory that contains the TAR1090 website files (typically `/usr/local/share/tar1090/html`) and open `layersxxxx.js`.
At the end of that file, right  before it says `return layers_group;` copy paste the following code:
```
        const aiscatcher_server = "http://192.168.1.113:8100";

        const aiscatcher_source = new ol.source.Vector({
            url: aiscatcher_server + '/geojson',
            format: new ol.format.GeoJSON()
        });

        function updateAIScatcher() {
            aiscatcher_source.refresh();
        }

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
            9: { size: [20, 20], offset: [0, 20], comment: 'CLASS_PLANE' },
            10: { size: [20, 20], offset: [0, 20], comment: 'CLASS_HELICOPTER' },
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
Note: make sure you refresh the javascript files (I do that by opening layersxxxx.js in the browser and refreshing). 

And then you have ships in TAR1090:
![Screenshot from 2024-01-06 22-34-53](https://github.com/jvde-github/AIS-in-TAR1090/assets/52420030/2b715abf-64f5-4cc6-8f4b-30e8661e077c)


