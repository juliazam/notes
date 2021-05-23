# Transpile SCSS-files into minified CSS-files 'on fly'

1. __Gulp__

Check is Gulp installed glogally:
```
gulp - v
```

If not, install it with the node package manager (__npm__):
```
npm i gulp -g
```

2. __package.json__

Create a folder for a project, step in and create a new __package.json__ file inside the project folder. To make this run the command below in the terminal: 
```
npm init
```
__package.json__ file helps __npm__ keep track of the other packages installed.

3. __Install packages__

All packages will be used only during development, thus not nescessary to install them globally. Use _--save-dev_ flag during installation.
* install __gulp__ locally as well
* install __del__ package to delete files
* install __gulp-concat__ package to gather all scss-files into one
* install __gulp-sass__ package to convert scss-files to css-files
* install __gulp-autoprefixer__ package to automatically prefix css
* install __gulp-clean-css__ package to minify css-files
* install __gulp-rename__ package to rename files
```
npm i gulp, del, gulp-concat, gulp-sass, gulp-autoprefixer, gulp-clean-css, gulp-rename --save-dev
```
4. __gulpfile.js__

I'm mostly use gulp tasks during Wordpress themes or plugins development. I have all my small \_*.scss files in _assets/scss/_  folder (this is my source folder), so before converting them into css-file, I have to have gather them in one _styles.sccs_ file. But if I already have _styles.scss_ file, I have to delete it first, because I didn't find how to overwrite the existing file, __gulp-sass__ always append it (if you know how to do that, please, let me know). 

Then I convert _styles.scss_ into _styles.css_ and save it in _assets/css/_ folder (this is my destination folder that then goes to production). The last step is to minify _styles.css_ to _styles.min.css_ (the file to enqueue in _functions.php_).

Usually, style files for the admin dashboard are not so huge, so I keep them in one _admin/styles_ folder.

I have _theme_ and _themeAdmin_ variabes to keep folder names for source and destination folders, thus, I can easyly change them if in need.
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

// Functions for transpiling scss-files to minified css-files
const themeCSS = series( deleteThemeFiles, concatThemeSCSS, convertThemeCSS, minifyThemeCSS );
const adminThemeCSS = series( convertThemeAdminCSS, minifyThemeAdminCSS );

// Function to track changes in  *.scss files and execute minification
function watching() {
    watch( [ theme.src + '*.scss' ], themeCSS );
    watch( [ themeAdmin.src + scssFile ], adminThemeCSS );
}

// Register task for public
exports.watching = watching;
```