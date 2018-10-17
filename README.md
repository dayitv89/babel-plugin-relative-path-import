# babel-plugin-relative-path-import

Babel plugin to add the opportunity to use `import` and `require` with root based paths.<br>
[![Build Status](https://travis-ci.org/dayitv89/babel-plugin-relative-path-import.svg?branch=master)](https://travis-ci.org/dayitv89/babel-plugin-relative-path-import)
[![Build Status](https://app.bitrise.io/app/233ca8f676ebaf7d/status.svg?token=3ZTN8OKoFWVzeHy085tG3A&branch=master)](https://app.bitrise.io/app/233ca8f676ebaf7d)
[![https://github.com/dayitv89/babel-plugin-relative-path-import](https://img.shields.io/npm/dm/babel-plugin-relative-path-import.svg)](https://www.npmjs.com/package/babel-plugin-relative-path-import)
![](https://img.shields.io/badge/Stable-v2.0.0-green.svg?style=flat)

## Versions:

- Babel 7 supports : `v2.0.0`
- Previous stable version: `v1.0.5`

## Example

```javascript
// Usually
import UserAction from '../../../actions/UserAction';
const Colors = require('../../theme/Colors');

// With babel-plugin-relative-path-import
import UserAction from '@actions/UserAction';
const Colors = require('@theme/Colors');
```

## Install

**npm**

```
npm install babel-plugin-relative-path-import --save-dev
```

**yarn**

```
yarn add babel-plugin-relative-path-import --dev
```

## Use

Add a `.babelrc` file and write:

```javascript
{
  "plugins": [
		[
			"babel-plugin-relative-path-import",
			{
				"paths": [{
					"rootPathPrefix": "~", // `@` is the default so you can remove this if you want
					"rootPathSuffix": "src/js"
				},
				{
					"rootPathPrefix": "@",
					"rootPathSuffix": "other-src/js"
				},
				{
					"rootPathPrefix": "@parent",
					"rootPathSuffix": "../../src/in/parent" // since we support relative paths you can also go into a parent directory
				},
				{
					"rootPathPrefix": "@some",
					"rootPathSuffix": "../../src/in/some" // since we support relative paths you can also go into a parent directory
				}]
			}
		]
	]
}

// Now you can use the plugin like:
import foo from '~/my-file';
const bar = require('@/my-file');

// Usually
import ParentExample from '../../src/in/parent/example.js';
const OtherExample = require('../../src/in/some/example.js');

// With Babel-Root-Importer
import ParentExample from '@parent/example.js';
const OtherExample = require('@some/example.js');
```

or pass the plugin with the plugins-flag on CLI

```
babel-node myfile.js --plugins babel-plugin-relative-path-import
```

### Don't let ESLint be confused

If you use [eslint-plugin-import](https://github.com/benmosher/eslint-plugin-import) to validate imports it may be necessary to instruct ESLint to parse root imports. You can use [eslint-import-resolver-babel-plugin-root-import](https://github.com/bingqichen/eslint-import-resolver-babel-plugin-root-import)

```json
    "import/resolver": {
      "babel-plugin-relative-path-import": {}
    }
```

### Don't let Flow be confused

If you use Facebook's [Flow](https://flowtype.org/) for type-checking it is necessary to instruct it on how to map your chosen prefix to the root directory. Add the following to your `.flowconfig` file, replacing `{rootPathPrefix}` with your chosen prefix and `{rootPathSuffix}` with your chosen suffix.

```
[options]
module.name_mapper='^{rootPathPrefix}/\(.*\)$' -> '<PROJECT_ROOT>/{rootPathSuffix}/\1'
```

## FYI

Webpack delivers a similar feature, if you just want to prevent end-less import strings you can also define `aliases` in the `resolve` module, at the moment it doesn't support custom/different symbols and multiple/custom suffixes.
[READ MORE](http://xabikos.com/2015/10/03/Webpack-aliases-and-relative-paths/)

### Want to revert back to relative paths?

Sometimes tooling might not be up to scratch, meaning you lose features such as navigation in your IDE. In such cases you might want to revert back to using relative paths again. If you have a significant amount of files, it might be worth looking into [tooling](https://www.npmjs.com/package/convert-root-import) to help you with the conversion.

### Distributing package?

While distributing package somewhere, `.babelrc` file has problem to resolve the path. To fix that you should run `postinstall` inside your package.json.

```js
// BabelPostScript.js
(function() {
	const path = process.cwd();
	if (path.includes('node_modules')) {
		const dataPrefix = path.substring(path.indexOf('node_modules')) + '/';
		const fs = require('fs');
		const config = {
			fileName: './.babelrc',
			fileType: 'utf8',
			dataPrefix
		};

		fs.readFile(config.fileName, config.fileType, function(err, data) {
			const bableDefault = JSON.parse(data);
			bableDefault.plugins[0][1].forEach(i => {
				i.rootPathSuffix = `${config.dataPrefix}${i.rootPathSuffix}`;
			});
			fs.writeFile(config.fileName, JSON.stringify(bableDefault, null, 2), function(err) {
				if (err) throw err;
				console.log('BabelPostScript for babel-plugin-relative-path-import run: complete');
			});
		});
	}
})();
```

Run this script using `postinstall` inside `package.json`

```js
{
	...
	"scripts": {
		...
		"postinstall": "node ./BabelPostScript.js"
	},
	...
}
```

### Credit

Improved by [Gaurav D. Sharma](https://github.com/dayitv89), inspired & originally taken from [entwicklerstube](https://github.com/entwicklerstube/babel-plugin-root-import)
