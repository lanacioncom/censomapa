# DATE=$(shell date +'%Y%m%d%H%M')
DATE=$(date +'%Y%m%d%H%M')
all:
	mkdir -p build
	rm -rf build/*
	# minificar javascript
	awk '/BEGIN_JS/ {flag=1;next} /END_JS/{flag=0} flag {print}' index.html | cut -d\" -f 2 | xargs cat | uglifyjs > build/all_$(DATE).js
	# procesar html y cambiar el path al js
	head -n `grep -n "BEGIN_JS" index.html | cut -d: -f 1` index.html > build/index.html.tmp
	echo "<script type=\"text/javascript\" src=\"all_$(DATE).js\"></script>" >> build/index.html.tmp
	tail -n +`grep -n "END_JS" index.html | cut -d: -f 1` index.html >> build/index.html.tmp
	# preprocesar css
	awk '/BEGIN_CSS/ {flag=1;next} /END_CSS/{flag=0} flag {print}' index.html | pcregrep -o1 'href="([^"]*)"' | xargs cat > /tmp/tmp.css
	# "sed -i -e" solo funciona en MacOS
	sed -i -e 's/\?cachebust/\?$(DATE)/' /tmp/tmp.css
	mkdir -p build/css
	csso /tmp/tmp.css > build/css/all_$(DATE).css
	rm /tmp/tmp.css
	# preprocesar html y cambiar el path al css
	head -n `grep -n "BEGIN_CSS" index.html | cut -d: -f 1` build/index.html.tmp > build/index.html
	echo '<link rel="stylesheet" href="css/all_$(DATE).css" type="text/css">' >> build/index.html
	tail -n +`grep -n "END_CSS" index.html | cut -d: -f 1` build/index.html.tmp >> build/index.html
	# "sed -i -e" solo funciona en MacOS
	sed -i -e 's/\?cachebust/\?$(DATE)/' build/index.html
	rm build/index.html-e
	rm build/index.html.tmp
	# copiar css y eso (preservando fechas para optmizar sync con el server)
	cp -RLp data build
	cp -RLp css/font build/css
	cp -RLp img build

TOPOJSON_QUANTIZATION=4000
TOPOJSON_SIMPLIFY_PROPORTION=.4
FUSION_TABLE_ID=12c7R5dNjevKmg6RHbOb9FJdGjo9olIAYUt99jF0
data:
	# genero mapa en topojson
	topojson -p DNE_ID -p PROVINCIA -p DEPARTAMEN --id-property DNE_ID -q $(TOPOJSON_QUANTIZATION) --simplify-proportion=$(TOPOJSON_SIMPLIFY_PROPORTION) -o /tmp/ea.json ../mapas/departamentos.shp ../mapas/provincias.shp
	# bajo la data del FT y la guardo en el formato que necesita el mapa
	python ../experimentos/export_data.py "$(FUSION_TABLE_ID)" > data/data.json
	# la bajo de vuelta en CSV para mergear con el topojson
	curl --header "accept-charset:ISO-8859-1,utf-8;q=0.7,*;q=0.3" -o /tmp/data.csv "https://www.google.com/fusiontables/exporttable?query=select+*+from+$(FUSION_TABLE_ID)"
	# mergear data y propiedades en el topojson
	python ../experimentos/merge_map_and_data.py /tmp/ea.json /tmp/data.csv > ../experimentos/topo/data/ea.json
	rm /tmp/ea.json
	rm /tmp/data.csv
