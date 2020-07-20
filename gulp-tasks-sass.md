1. Create package.json file. It helps the node package manager (npm) keep track of the npm packages installed. 
```
npm init -y
```
*Option -y used to skip the questionnaire altogether.*

2. Install Gulp and Gulp modules to create tasks that will automate SASS->CSS transpilation.

..* Check if node-sass module installed. This module transpile *.scss into *.css files.
```
node-sass -v
```

If not, install it globally (with -g option):
```
npm install -g node-sass
```

..* Install Gulp (globally) itself.
```
npm install -g gulp
```
And locally for development:
```
npm i --save-dev gulp
```

..* Install gulp-sass locally for development invironment.
```
npm i --save-dev gulp-sass
```

..* To connect small *.scss files to one install gulp-concat for development invironment.
```
npm i --save-dev gulp-concat
```

..* To have clean *.css files (minified) and with right prefixes install next modules for development invironment.
```
npm i --save-dev gulp-clean-css gulp-autoprefixer
```

..* To rename and delete files install gulp-rename and del modules. As development dependencies as well.
```
npm i --save-dev gulp-rename del
```
*All commands in one*
```
npm i --save-dev gulp gulp-sass gulp-concat gulp-clean-css gulp-autoprefixer gulp-rename del
```

Check package.json file to make sure that all development dependencies are installed.

3. Create gulpfile.js file in the root of project. At the beginning of file configure variables.
```
const gulp = require('gulp');
const concatSASS = require('gulp-concat');
const convertSASS = require('gulp-sass');
const minifyCSS = require('gulp-clean-css');

const autoprefixer = require('gulp-autoprefixer');

const rename = require('gulp-rename');
const deletefile = require('del');
```

... I keep *.scss files in assets/scss folder, and *.css files in assets/css folder. I concat *.scss files into styles.scss file, then I compile it into styles.css file, then minify it as styles.min.css. Thus I had to remove them first, before transpilation.

*If all these files kept in the root of project, not necessary to delete files before, besause there is an option to overwrite them.*

```
// Delete style files before transpile
gulp.task('delete-files', function() {
    return deletefile(['assets/scss/styles.scss', 'assets/css/styles.css', 'assets/css/styles.min.css']);
});
```

... Then connect all *.scss files in styles.scss file.
```
// Compile all *.scss into one styles.scss
gulp.task('concat-sass', function() {
  return gulp.src('assets/scss/*.scss')
    .pipe(concatSASS('styles.scss'))
    .pipe(gulp.dest('./assets/scss/'));
});
```
*Note: a glob is a string of literal and/or wildcard characters, like '*', '**', or '!', used to match filepaths. Globbing is the act of locating files on a file system using one or more globs. In the case above gulp uses the path 'assets/scss/' as a glob base. As {base:'.'} option doesn't specified, the glob base used to form destination folder.

With {base:'.'} option, the destination file will be created in the root folder, and it will be overwritten. Otherwize, the append mode will be used.*
```
// Compile all *.scss into one styles.scss
gulp.task('concat-sass', function() {
  return gulp.src('assets/scss/*.scss', {base:'.'})
    .pipe(concatSASS('styles.scss'))
    .pipe(gulp.dest('./assets/scss/'));
});
```

... Then transpile styles.scss file into styles.css:
```
// Transpile *.scss files to *.css
gulp.task('convert-sass', function() {
  return gulp.src('assets/scss/styles.scss')
    .pipe(convertSASS())
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('assets/css'));
});
```
... After that minify css file.
```
// Minify *.css files
gulp.task('minify-css', function() {
  return gulp.src('assets/css/styles.css')
    .pipe(rename('styles.min.css'))
    .pipe(minifyCSS({compatibility: 'ie8'}))
    .pipe(gulp.dest('assets/css/'));
});
```
... While the tasks above may be used one by one, the task below perform all tasks together.
```
// Compile->transpile->minify, then watch changes
gulp.task('convert-all-once',  gulp.series(['delete-files', 'concat-sass', 'convert-sass', 'minify-css']), function(done) {
  done();
});
```
... The task below allows to watch for scss files, and compile-trinspile-minify files, when scss file was changed.
```
gulp.task('watch', gulp.series(['concat-sass', 'convert-sass', 'minify-css'], function(done) {
  gulp.watch('assets/scss/*.scss', gulp.series(['delete-files', 'concat-sass', 'convert-sass', 'minify-css']));
  done();
}));
```