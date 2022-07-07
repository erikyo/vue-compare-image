<template>
  loading: {{ loading }}
</template>

<script>
export default {
  name: "DataParser",
  data() {
    return {
      loading: true,
      sourceImage: {},
    }
  },
  methods: {
    async fetchData(imageDirectory){
      return await fetch( "tests/" + imageDirectory + "/output.txt", {mode: 'no-cors'})
          .then(res => res.text())
          .catch(error => console.log(error) /* Handle the error response object*/
          );
    },

    initCompare(ImagesPath = 'jpeg75'){

      // step 1 - get the file and detect the initial data
      this.fetchData(ImagesPath)

        .then( (data) => {
          this.loading = "⌛ Parsing"

          // step 2 - get the raw file
          return this.ParseRaw(data);

        })
        .then((parsedData) => {

          this.loading = "🎉 Done!"

          console.log("🎉 Done!")

          this.$root.$data.imagesData = parsedData;

          this.$root.$data.originalImage = parsedData[this.sourceImage.source];
          this.$root.$data.referenceImage = parsedData[this.sourceImage.reference];

          this.$root.$data.width = parsedData[this.sourceImage.source].width;
          this.$root.$data.height = parsedData[this.sourceImage.source].height;

          return {
            left: parsedData[this.sourceImage.source],
            right: parsedData[this.sourceImage.reference]
          };

        })
        .then((currentComparison) => {

          this.loading = false

          this.$root.$data.currentCompare = currentComparison;
          this.$emit('getResult', currentComparison)

        });
    },

    formatExtension(ext) {
      if (ext === 'pjpg') return 'Progressive JPEG'
      if (ext === 'mozjpg') return 'MozJpeg'
      if (ext === 'jpg') return 'Jpeg'
      else return ext
    },

    parseIdentify(data) {
      const raw = data.split(" ");
      return {
        directory: raw[0],
        basename: raw[1],
        extension: raw[2],
        filename: raw[1] + "." + raw[2],
        compression: raw[3],
        channels: raw[4],
        sizebyte: parseInt(raw[5], 10),
        filesize: this.$root.$options.methods.humanFileSize(raw[5]),
        width: parseInt(raw[6], 10),
        height: parseInt(raw[7], 10),
        estimatedQuality: parseInt(raw[8], 10)
      }
    },

    extractImageAnalysis(raw, line) {
      let result = {};
      if (raw[line+4].trim().startsWith("alpha:")){
        result.alpha = raw[line+4].trim().replace("alpha: ", "")
        result.all = raw[line+5].trim().replace("all: ", "")
      } else {
        result.all = raw[line+4].trim().replace("all: ", "")
      }
      result.red = raw[line+1].trim().replace("red: ", "")
      result.green = raw[line+2].trim().replace("green: ", "")
      result.blue = raw[line+3].trim().replace("blue: ", "")
      return result;
    },

    ParseRawHeader(raw) {
      this.$root.$data.path = "tests/" + raw[1].replace("SOURCEPATH: ", '');
      return {
        source: raw[2].replace("SOURCEFILE: ", ''),
        reference: raw[3].replace("REFERENCEFILE: ", '')
      }
    },

    ParseRaw(rawData) {

      let parsed = rawData.split("PROCESSING: ");
      let result = {};

      this.sourceImage = this.ParseRawHeader(parsed[0].split("\n"));

      delete parsed[0];

      console.log(this.sourceImage);

      Object.values( parsed ).forEach( image => {

        // split data for each line
        image = image.split("\n");

        // the filename without the extension
        const name = image[0].split(".")[0];

        // the base filedata
        result[image[0]] = {
          name: name,
          type: name.split("-")[1] || name + " " + image[0].split(".")[1],
          quality: parseInt(name.split("-")[2], 10) || 100,
        };

        result[image[0]].formattedType = this.formatExtension(result[image[0]].type);

        // for each line search if starts for a string,
        // this string identifies the type of data in the following line
        image.forEach((imgData, index) => {

          if (imgData.indexOf("STATS:") !== -1) {

            var stats = imgData.replace("STATS:", '', imgData).split(",")
            stats.forEach((stat) => {
              result[image[0]][stat.split(" ")[1]] = stat.split(" ")[0]
            })

          } else if (imgData.indexOf("TIMETOCOMPLETE:") !== -1) {

            result[image[0]].encodetime = imgData.replace("TIMETOCOMPLETE:", '', imgData)

          } else if (imgData.indexOf("RESULT:") !== -1) {

            var imageData = this.parseIdentify(imgData.replace("RESULT:", ''))
            for (var attrname in imageData) {
              result[image[0]][attrname] = imageData[attrname];
            }

          } else if (imgData.indexOf("  Channel distortion: SSIM") !== -1) {

            result[image[0]].ssim = this.extractImageAnalysis(image, index);
            result[image[0]].similarity = result[image[0]].ssim.all

          } else if (imgData.indexOf("  Channel distortion: DSSIM") !== -1) {

            result[image[0]].dssim = this.extractImageAnalysis(image, index);

          } else if (imgData.indexOf("  Channel distortion: PSNR") !== -1) {

            result[image[0]].psnr = this.extractImageAnalysis(image, index);
            result[image[0]].noise = result[image[0]].psnr.all

          } else if (imgData.indexOf("  Channel distortion: MAE") !== -1) {

            result[image[0]].mae = this.extractImageAnalysis(image, index);

          }
        })

        // Since some data as the source file doesn't have some data (eg. encode time),
        // we fill that attribute with some fake data
        if (!result[image[0]].similarity) result[image[0]].similarity = 1
        if (!result[image[0]].noise) result[image[0]].noise = 0
        if (!result[image[0]].encodetime) result[image[0]].encodetime = 0.00001
        if (!result[image[0]].cpu) result[image[0]].cpu = 100
      })

      return result;
    }
  },
  created: function () {
    this.loading = true
    this.initCompare()
  }
}
</script>