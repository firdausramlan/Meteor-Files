##### Create and Manage File's subversion

```javascript
this.Videos = new FilesCollection({
  /* .. other options .. */
  collectionName: 'Videos',
  onAfterUpload: function(fileRaf) {
    var formats, sourceFile;
    sourceFile = ffmpeg(fileRef.path).noProfile();
    formats = {
      ogg: true,
      mp4: true,
      webm: true
    };
    _.each(formats, function(convert, format) {
      var file, upd, version;
      if (convert) {
        file = _.clone(sourceFile);
        version = file.someHowConvertVideoAndReturnFileData(format);
        upd = {
          $set: {}
        };
        upd['$set']['versions.' + format] = {
          path: version.path,
          size: version.size,
          type: version.type,
          extension: version.extension
        };
        return Videos.update(fileRef._id, upd);
      }
    });
  }
});

if (Meteor.isClient) {
  Template.upload.events({
    'change #upload': function(e, template) {
      /* Upload all Files */
      _.each(e.currentTarget.files, function(file) {
        Videos.insert({
          file: file,
          onUploaded: function(error, fileObj) {
            if (error) {
              alert(error.message);
              throw new Meteor.Error 500, error;
            }
          },
          onBeforeUpload: function() {
            if (['ogg', 'mp4', 'avi', 'webm'].inArray(this.ext) && this.size < 512 * 1048 * 1048) {
              return true;
            } else {
              return "Please upload file in next formats: 'ogg', 'mp4', 'avi', 'webm' with size less than 512 Mb. You have tried to upload file with \"" + this.ext + "\" extension and with \"" + (Math.round((this.size / (1024 * 1024)) * 100) / 100) + "\" Mb";
            }
          },
          streams: 8
        });
      });
    }
  });
}
```