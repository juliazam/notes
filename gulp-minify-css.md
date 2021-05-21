# Transpile SCSS-files into minified CSS-files 'on fly'

1. __Node.js and Gulp__

Check both are installed glogally:
```
node -v
```
```
gulp - v
```

If not, install them with the node package manager (__npm__):
```
npm i node -g
```
```
npm i gulp -g
```

2. __package.json__

Create a __package.json__ file for the project. It helps __npm__ keep track of the other packages installed.
```
npm init
```

3. __Install packages__

All packages will be used only during development, thus not nescessary to install them globally. Use _--save-dev_ flag during installation.
* install __gulp__ locally
* install __del__ package to delete files
* install __gulp-concat__ package to gather all scss-files into one
* install __gulp-sass__ package to convert scss-files to css-files
* install __gulp-autoprefixer__ package to autonatically prefix css
* install __gulp-clean-css__ package to minify css-files
* install __gulp-rename__ packahe to rename files
```
npm i gulp, del, gulp-concat, gulp-sass, gulp-autoprefixer, gulp-clean-css, gulp-rename --save-dev
```
4. __gulpfile.js__

```
const { src, dest, watch, series }  = require( 'gulp' );

// All required modules for transpiling scss-files to minifiled css-files
const   del = require( "del" ),
        concat = require( 'gulp-concat' ),
        scss2css = require( 'gulp-sass' ),
        autoprefixer = require( 'gulp-autoprefixer' ),
        minify = require( 'gulp-clean-css' ),
        rename = require( 'gulp-rename' );

// Style files folders: for theme, admin dashboard and block editor
const theme = {
    src: "assets/scss/",
    dest: "assets/css/"
},
themeAdmin = {
    src: "admin/styles/",
    dest: "admin/styles/"
},
blockEditor = {
    src: "block-editor/styles/",
    dest: "block-editor/styles/"
},
blockEditorAdmin = {
    src: "block-editor/admin/styles/",
    dest: "block-editor/admin/styles/"
};

// Style files names
const   scssFile = 'styles.scss',
        cssFile = 'styles.css',
        minified = 'styles.min.css'

// Delete files
function deleteFiles( obj ) {
    return del( obj.src + scssFile, obj.dest + '*.css' );
}
function deleteThemeFiles() {
    return deleteFiles( theme );
}

// Concatenate _*.scss files to  one scss-file
function concatSCSS( obj ) {
    return src( obj.src + '*.scss' )
            .pipe( concat( scssFile ) )
            .pipe( dest( obj.src ) );
}
function concatThemeSCSS() {
    return concatSCSS( theme );
}

// Convert scss to css
function convert2CSS( obj ) {
    return src( obj.src + scssFile )
            .pipe( scss2css() )
            .pipe( autoprefixer( 'last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4' ) )
            .pipe( dest( obj.dest ) );
}
function convertThemeCSS() {
    return convert2CSS( theme );
}
function convertThemeAdminCSS() {
    return convert2CSS( themeAdmin );
}
function convertBlockEditorCSS() {
    return convert2CSS( blockEditor );
}
function convertBlockEditorAdminCSS() {
    return convert2CSS( blockEditorAdmin );
}

// Minify CSS
function minifyCSS( obj ) {
    return src( obj.dest + cssFile )
            .pipe( rename( minified ) )
            .pipe( minify( {compatibility: 'ie8'} ) )
            .pipe( dest( obj.dest ) );
}
function minifyThemeCSS() {    
    return minifyCSS( theme );
}
function minifyThemeAdminCSS() {
    return minifyCSS( themeAdmin );
}
function minifyBlockEditorCSS() {
    return minifyCSS( blockEditor );
}
function minifyBlockEditorAdminCSS() {
    return minifyCSS( blockEditorAdmin );
}

// Functions for transpiling scss-files to minified css-files
const themeCSS = series( deleteThemeFiles, concatThemeSCSS, convertThemeCSS, minifyThemeCSS );
const adminThemeCSS = series( convertThemeAdminCSS, minifyThemeAdminCSS );
const blockEditorCSS = series( convertBlockEditorCSS, minifyBlockEditorCSS );
const blockEditorAdminCSS = series( convertBlockEditorAdminCSS, minifyBlockEditorAdminCSS );

// Function to track changes in  *.scss files and execute minification
function watching() {
    watch( [ theme.src + '*.scss' ], themeCSS );
    watch( [ themeAdmin.src + scssFile ], adminThemeCSS );
    watch( [ blockEditor.src + scssFile ], blockEditorCSS );
    watch( [ blockEditorAdmin.src + scssFile ], blockEditorAdminCSS );
}

// Register task for public
exports.watching = watching;
```