# Flr Core Logic Document

## 前言

这是`Flr`的核心逻辑描述文档，用于`Flr`的核心功能的逻辑细节，以确保`Flr`工具系列版本（[CLI版本](https://github.com/Fly-Mix/flr-cli)、[VSCode Extension版本](https://github.com/Fly-Mix/flr-vscode-extension)、[Android Studio Plugin版本](https://github.com/Fly-Mix/flr-as-plugin)）在核心功能上能保持良好的一致性。而更进一步地，我们希望未来能在其他输出产物（文件输出、日志输出、交互输出）上也保持一致性。



## Flr术语

#### 版本号

1. Version （工具版本号）：指具体实现了Flr功能的工具（如flr-cli）的版本号，由各工具开发者自己管理
2. Core Logic Version（核心逻辑版本号）：指Flr工具所使用的核心逻辑的版本号（如第一个核心逻辑的版本号是`1.0.0`），由开发团队统一管理

#### 资源

1. `resource_dir`: the resource directory relative to flutter project's root directory, for example: `lib/assets/images`, `lib/assets/jsons` .
1. `implied_resource_dir`:  the `resource_dir` without `lib/`, for example: `assets/images`, `assets/jsons` .
1. `package_name`: the name of flutter project's package production, for example, the `package_name` of [Flutter-R Demo Project](https://github.com/Fly-Mix/flutter_r_demo) is `flutter_r_demo`. It can be accessed from `pubspec.yaml`.
1. `resource_file`: the real file in resource directory , for example, the resource files in `lib/assets/images` resource directory:
   - `lib/assets/images/hot_foot_N.png` 
   - `lib/assets/images/3.0x/hot_foot_N.png`
1. `implied_resource_file`:  the `resource_file` without `lib/`, for example: `assets/images/hot_foot_N.png` .
1. `file_basename`: the resource file's basename, for example: `hot_foot_N.png` .
1. `file_basename_no_extension`: the resource file's basename with no file extension, for example: `hot_foot_N` .
1. `file_extname`: the resource file's extension, for example: `.png`
1. `file_dirname`: the resource file's dirname, for example: `lib/assets/images`
1. `asset`: the resource file's mapping file in flutter project's package production, also the resource file's declaration in `pubspec.yaml`. For example: `packages/flutter_r_demo/assets/images/hot_foot_N.png` , `packages/flutter_r_demo/assets/jsons/test.json` .
1. `asset_name`: The name of the main `asset` from the set of asset resources to choose from. For example: `assets/images/hot_foot_N.png` , `assets/jsons/test.json` .
1. `image_asset`: the mapping and declaration of image resource file. It's definition is: `packages/#{package_name}/#{implied_resource_dir}/#{file_basename}` . For example: `packages/flutter_r_demo/assets/images/hot_foot_N.png` .  Each `image_asset` correspond to one main `resource_file` and it's variants, for example, `packages/flutter_r_demo/assets/images/hot_foot_N.png` correspond to `lib/assets/images/hot_foot_N.png` and it's variants (such as `lib/assets/images/2.0x/hot_foot_N.png`) .
1. `text_asset`: the mapping and declaration of text resource file. It's definition is: `packages/#{package_name}/#{implied_resource_file}`. For example: `packages/flutter_r_demo/assets/jsons/test.json` . Each `text_asset` correspond to one  `resource_file`, for example `packages/flutter_r_demo/assets/jsons/test.json` correspond to `lib/assets/jsons/test.json` .
1. `font_asset`: the mapping and declaration of text resource file. It's definition is: `packages/#{package_name}/#{implied_resource_file}`. For example: `packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Regular.ttf` . Each `text_asset` correspond to one  `resource_file`, for example `packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Regular.ttf` correspond to `lib/assets/fonts/Amiri/Amiri-Regular.ttf` .
1. `asset_id`: the identity of `asset`  , which is usually `file_basename_no_extension` of  `asset`. 
1. `image_file`: includes `non-svg_image_file` and `svg_image_file` .
   - `non-svg_image_file`: the resource file with suffixes:  `.png`,  `.jpg`,  `.jpeg`, `.gif`, `.webp`, `.icon`, `.bmp`, `.wbmp` .
   - `svg_image_file`: the resource file with suffixes:  `.svg` .
1. `text_file`: the resource file with suffixes: `.txt`、`.json`、`.yaml`、`.xml` .
1. `font_file`: the resource file with suffixes: `.ttf`、`.otf`、`.ttc`.
1. `font_family`: the family  of  `font_file`.
1. `font_family_id`: the identity of `font_family` , which is usually  name of `font_family` .

> - 以上有部分定义参考自 [Visual Studio Code](https://code.visualstudio.com/docs/editor/variables-reference)
> - 以上命名规则使用的是Ruby的规则，在实际实现时，请优先使用所选的开发语言的命名规则

#### 合法性

1. `legal_resource_file`: the `file_basename_no_extension` of  `resource_file` is made up of  letters (`a-z`, `A-Z`), numbers (`0-9`) and other legal characters (`_`, `+`, `-`, `.`, `·`, `!`, `@`, `&`, `$`, `￥`) .
2. `illegal_resource_file`: bad `legal_resource_file` .
3. `legal_resource_dir`: the `resource_dir` does exist in flutter project.
4. `illegal_resource_dir`: bad `legal_resource_dir` .

#### 一致性

1. flr核心逻辑一致：指`Flr配置`和当前Flr工具的`Core Logic Version`的值相等。

#### 配置

1. `flr_config`: the configuration of `Flr` which is specified in `pubspec.yaml` by `Flr`. 

   ```yaml
   flr:
     core_version: 1.0.0
     dartfmt-line-length: 80
     assets: []
     fonts: []
   ```

   -  `core_version`:  用于存放`Flr`工具实现的核心逻辑版本（`CoreLogic version`），初始值由`Flr`工具执行初始化时自动填写

   -  `dartfmt-line-length`：用于存放`Flr`的CLI版本工具对`r.g.dart`进行格式化时的行长，初始值为：80（dartfmt工具的默认值）；后期用户可根据团队需要进行手动修改

   - `assets`：用于配置需要`Flr`扫描图片资源和文本资源的`resource_dir`数组，初始值为空数组`[]`；初始化完毕后，新值由用户填写，如：

     ```yaml
     - lib/assets/images
     - lib/assets/texts
     ```

   - `fonts`：用于配置需要`Flr`扫描字体资源的`resource_dir`数组，初始值为空数组`[]`；初始化完毕后，新值由用户填写，如：

      ```yaml
      - lib/assets/fonts
      ```

2. `flutter_assets_config`: the configuration of assets of flutter which is an asset array, such as:

   ``` yaml
   - packages/flutter_r_demo/assets/images/test.png
   - packages/flutter_r_demo/assets/images/test.svg
   - packages/flutter_r_demo/assets/jsons/test.json
   - packages/flutter_r_demo/assets/yamls/test.yaml
   ```

3. `flutter_fonts_config`: the configuration of fonts of flutter which is an `font_family_config` array, such as:

   ```yaml
   - family: Amiri
     fonts:
       - asset: packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Regular.ttf
       - asset: packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Italic.ttf
   - family: Baloo_Thambi_2
     fonts:
       - asset: packages/flutter_r_demo/assets/fonts/Open_Sans/OpenSans-Regular.ttf    
   ```

4. `font_family_config`: the configuration of a font family which is an map, such as:

   ```
   family: Amiri
   fonts:
     - asset: packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Regular.ttf
     - asset: packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Italic.ttf
   ```

5. `font_asset_config`: the configuration of each font asset entry in`font_family_config`, such as:

   ```yaml
   asset: packages/flutter_r_demo/assets/fonts/Amiri/Amiri-Regular.ttf
   ```

   

## Flr推荐的和强制的资源目录组织结构

为了让`Flr`更好地扫描资源，flutter开发者在flutter工程应该（SHOULD）按照以下建议组织整体资源目录结构，以及应该（SHOULD）按照以下建议组织存放图片资源和文本资源，必须（MUST）按照以下要求组织存放字体资源。

#### Flr推荐的整体资源目录组织结构

flutter开发者在flutter工程应该（SHOULD）按照以下建议组织整体资源目录结构，包括组织图片资源、文本资源、字体资源：

```
flutter_project_root_dir
├── build
│   ├── ..
├── lib
│   ├── assets // 资源总目录
│   │   ├── images // 图片资源总目录
│   │   │   ├── test.png
│   │   │   ├── 2.0x
│   │   │   │   ├── test.png
│   │   │   │   └── test.png
│   │   ├── texts // 文本资源总目录
│   │   │   └── test.json
│   │   │   └── test.yaml
│   │   ├── fonts // 字体资源总目录
│   ├── main.dart
│   ├── ..
```

#### Flr推荐的图片资源和文本资源目录组织结构

flutter开发者在flutter工程应该（SHOULD）按照以下建议组织结构存放图片资源和文本资源——其中，若某类资源较多，如图片资源，建议按照模块分类存放；其中，若某类资源较少，如文本资源，可直接存放在一个文件夹即可。

```
flutter_project_root_dir
├── build
│   ├── ..
├── path
│   ├── to
│   │   ├── moduleA_images // 模块A的图片资源总目录
│   │   │   ├── testA.png
│   │   │   ├── 2.0x
│   │   │   │   ├── testA.png
│   │   │   │   └── testA.png
│   │   ├── moduleB_images // 模块B的图片资源总目录
│   │   │   ├── testB.png
│   │   │   ├── 2.0x
│   │   │   │   ├── testB.png
│   │   │   │   └── testB.png
│   │   ├── texts // 文本资源总目录
│   │   │   └── test.json
│   │   │   └── test.yaml
│   ├── ..
```

#### Flr强制的字体资源目录组织结构

为了让`Flr`能正确完成字体资源的扫描、声明、代码生成的操作，flutter开发者必须按照以下组织结构存放文本资源：

```
flutter_project_root_dir
├── build
│   ├── ..
├── path
│   ├── to
│   │   ├── fonts // 字体资源目录
│   │   │   ├── #{font_family_name} // 某个字体的家族名称
│   │   │   │   ├── #{font_family_name}-#{font_weight_or_style}.ttf
│   │   │   ├── Amiri
│   │   │   │   ├── Amiri-Regular.ttf
│   │   │   │   ├── Amiri-Bold.ttf
│   │   │   │   ├── Amiri-Italic.ttf
│   │   │   │   ├── Amiri-BoldItalic.ttf
│   ├── ..
```




## Flr 核心逻辑

`Flr`包括4个核心功能：

- `init`: 初始化项目。
- `generate`: 根据配置扫描资源目录，为合法的资源添加声明以及生成`r.g.dart`。
- `start monitor`: 启动资源变化监控服务，以监控到有符合条件的资源变化时，调用`generate`功能。
- `stop monitor`: 停止资源变化监控服务。

下面将会逐一列出这4个核心功能的核心逻辑。

#### init

1. 进行环境检测；若发现不合法的环境，则抛出异常，终止当前进程。

    - 检测当前flutter工程根目录是否存在`pubspec.yaml`。

1. 添加`flr_conig`和[r_dart_library](https://github.com/YK-Unit/r_dart_library)的依赖声明到`pubspec.yaml`。

    - 添加`flr_conig`到`pubspec.yaml`。
       - 添加[r_dart_library](https://github.com/YK-Unit/r_dart_library)的库依赖声明到`pubspec.yaml`。
       - 根据[《`r_dart_library`-`FlutterSDK` 依赖关系表 》](https://github.com/YK-Unit/r_dart_library#dependency-relationship-table)和当前`FlutterSDK`的版本获取当前工程需要的最新依赖版本，如`0.1.1` 。
       - 添加库依赖声明到`pubspec.yaml`。

       ``` yaml
       r_dart_library:
         git:
           url: "https://github.com/YK-Unit/r_dart_library.git"
           ref: 0.1.1
       ```

1. 对Flutter配置进行修正，以避免执行获取依赖操作时会失败：

    - 检测Flutter配置中的`assets`选项是否是一个非空数组；若不是，则删除`assets`选项；
    - 检测Flutter配置中的`fonts`选项是否是一个非空数组；若不是，则删除`fonts`选项。

1. 调用flutter工具，为flutter工程获取依赖。

#### generate

1. 进行环境检测；若发现不合法的环境，则抛出异常，终止当前进程：

   - 检测当前flutter工程根目录是否存在`pubspec.yaml`。

   - 检测当前`pubspec.yaml中`是否存在`flr_conig`。

   - 检测当前`flr_config`中的`resource_dir`配置是否合法：

      判断合法的标准是：`assets`配置或者`fonts`配置了至少1个`legal_resource_dir`。

2. 进行核心逻辑版本检测：

   检测`flr_config`中的`core_version`和当前工具的`core_version`是否一致；若不一致，则生成“核心逻辑版本不一致”的警告日志，存放到警告日志数组。

3. 获取`assets_legal_resource_dir`数组、`fonts_legal_resource_dir`数组和`illegal_resource_dir`数组：

   - 从`flr_config`中的`assets`配置获取`assets_legal_resource_dir`数组和`assets_illegal_resource_dir`数组；
   - 从`flr_config`中的`fonts`配置获取`fonts_legal_resource_dir`数组和`fonts_illegal_resource_dir`数组；
   - 合并`assets_illegal_resource_dir`数组和`fonts_illegal_resource_dir`数组为`illegal_resource_dir`数组‘；若`illegal_resource_dir`数组长度大于0，则生成“存在非法的资源目录”的警告日志，存放到警告日志数组。

4. 扫描`assets_legal_resource_dir`数组中的`legal_resource_dir`，输出`image_asset`数组和`illegal_image_file`数组：

   - 创建`image_asset`数组、`illegal_image_file`数组；

   - 遍历`assets_legal_resource_dir`数组，按照如下处理每个资源目录：
      - 扫描当前资源目录和其第1级的子目录，查找所有`image_file`；
      - 根据`legal_resource_file`的标准，筛选查找结果生成`legal_image_file`子数组和`illegal_image_file`子数组；`illegal_image_file`子数组合并到`illegal_image_file`数组；
      - 根据`image_asset`的定义，遍历`legal_image_file`子数组，生成`image_asset`子数；组；`image_asset`子数组合并到`image_asset`数组。
   - 对`image_asset`数组做去重处理；
   - 按照字典顺序对`image_asset`数组做升序排列（一般使用开发语言提供的默认的sort算法即可）；
   - 输出`image_asset`数组和`illegal_image_file`数组。

5. 扫描`assets_legal_resource_dir`数组中的`legal_resource_dir`，输出`text_asset`数组和`illegal_text_file`数组：

   - 创建`text_asset`数组、`illegal_text_file`数组；

   - 遍历`assets_legal_resource_dir`数组，按照如下处理每个资源目录：
      - 扫描当前资源目录和其所有层级的子目录，查找所有`text_file`；
      - 根据`legal_resource_file`的标准，筛选查找结果生成`legal_text_file`子数组和`illegal_text_file`子数组；`illegal_text_file`子数组合并到`illegal_text_file`数组；
      - 根据`text_asset`的定义，遍历`legal_text_file`子数组，生成`text_asset`子数组；`text_asset`子数组合并到`text_asset`数组。
   - 对`text_asset`数组做去重处理；
   - 按照字典顺序对`text_asset`数组做升序排列（一般使用开发语言提供的默认的sort算法即可）；
   - 输出`text_asset`数组和`illegal_image_file`数组。

6. 扫描`fonts_legal_resource_dir`数组中的`legal_resource_dir`，输出`font_family_config`数组、`illegal_font_file`数组：

   - 创建`font_family_config`数组、`illegal_font_file`数组；
   - 遍历`fonts_legal_resource_dir`数组，按照如下处理每个资源目录：
      - 扫描当前资源目录，获得其第1级子目录数组，并按照字典顺序对数组做升序排列（一般使用开发语言提供的默认的sort算法即可）；
      - 遍历第1级子目录数组，按照如下处理每个子目录：
         - 获取当前子目录的名称，生成`font_family_name`；
         - 扫描当前子目录和其所有子目录，查找所有`font_file`；
         - 根据`legal_resource_file`的标准，筛选查找结果生成`legal_font_file`数组和`illegal_font_file`子数组；`illegal_font_file`子数组合并到`illegal_font_file`数组；
         - 据`font_asset`的定义，遍历`legal_font_file`数组，生成`font_asset_config`数组；
         - 按照字典顺序对生成font_asset_config数组做升序排列（比较asset的值）；
         - 根据`font_family_config`的定义，为当前子目录组织`font_family_name`和`font_asset_config`数组生成`font_family_config`对象，添加到`font_family_config`子数组；`font_family_config`子数组合并到`font_family_config`数组。
   - 输出`font_family_config`数组、`illegal_font_file`数组；
   - 按照字典顺序对font_family_config数组做升序排列（比较family的值）。

7. 检测是否存在`illegal_resource_file`：

   - 合并`illegal_image_file`数组、`illegal_text_file`数组和`illegal_font_file`数组为`illegal_resource_file`数组；
   - 若`illegal_resource_file`数组长度大于0，则生成“存在非法的资源文件”的警告日志，存放到警告日志数组。

8. 为扫描得到的`legal_resource_file`添加资源声明到`pubspec.yaml`：

   - 合并`image_asset`数组和`text_asset`数组为`asset`数组（`image_asset`数组元素在前）;
   - 修改`pubspec.yaml`中`flutter-assets`配置的值为`asset`数组；
   - 修改`pubspec.yaml`中`flutter-fonts`配置的值为`font_family_config`数组。

9. 按照SVG分类，从`image_asset`数组筛选得到有序的`non_svg_image_asset`数组和`svg_image_asset`数组：

   - 按照SVG分类，从`image_asset`数组筛选得到`non_svg_image_asset`数组和`svg_image_asset`数组；
   - 按照字典顺序对`non_svg_image_asset`数组和`svg_image_asset`数组做升序排列（一般使用开发语言提供的默认的sort算法即可）；

10. 在当前根目录下创建新的`r.g.dart`文件。

11. 根据下面模板生成`R` 类的代码，追加写入`r.g.dart`。（注意，模板中类似`#{value}`的为模板变量，如`#{package_name}`）

    - `#{package_name}`：Flutter工程包名，可从`pubspec.yaml`中读取

    ``` dart
    // IT IS GENERATED BY FLR - DO NOT MODIFY BY HAND
    // YOU CAN GET MORE DETAILS ABOUT FLR FROM:
    // - https://github.com/Fly-Mix/flr-cli
    // - https://github.com/Fly-Mix/flr-vscode-extension
    // - https://github.com/Fly-Mix/flr-as-plugin
    //
    
    // ignore: unused_import
    import 'package:flutter/widgets.dart';
    // ignore: unused_import
    import 'package:flutter/services.dart' show rootBundle;
    // ignore: unused_import
    import 'package:path/path.dart' as path;
    // ignore: unused_import
    import 'package:flutter_svg/flutter_svg.dart';
    // ignore: unused_import
    import 'package:r_dart_library/asset_svg.dart';
    
    /// This `R` class is generated and contains references to static asset resources.
    class R {
      /// package name: #{package_name}
      static const package = "#{package_name}";
    
      /// This `R.image` struct is generated, and contains static references to static non-svg type image asset resources.
      static const image = _R_Image();
    
      /// This `R.svg` struct is generated, and contains static references to static svg type image asset resources.
      static const svg = _R_Svg();
    
      /// This `R.text` struct is generated, and contains static references to static text asset resources.
      static const text = _R_Text();
      
      /// This `R.fontFamily` struct is generated, and contains static references to static font asset resources.
      static const fontFamily = _R_FontFamily();
    }
    ```

12. 根据下面模板生成`AssetResource `类的代码，追加写入`r.g.dart`。

    ``` dart
    /// Asset resource’s metadata class.
    /// For example, here is the metadata of `packages/flutter_demo/assets/images/example.png` asset:
    /// - packageName：flutter_demo
    /// - assetName：assets/images/example.png
    /// - fileDirname：assets/images
    /// - fileBasename：example.png
    /// - fileBasenameNoExtension：example
    /// - fileExtname：.png
    class AssetResource {
      /// Creates an object to hold the asset resource’s metadata.
      const AssetResource(this.assetName, {this.packageName}) : assert(assetName != null);
    
      /// The name of the main asset from the set of asset resources to choose from.
      final String assetName;
    
      /// The name of the package from which the asset resource is included.
      final String packageName;
    
      /// The name used to generate the key to obtain the asset resource. For local assets
      /// this is [assetName], and for assets from packages the [assetName] is
      /// prefixed 'packages/<package_name>/'.
      String get keyName => packageName == null ? assetName : "packages/$packageName/$assetName";
    
      /// The file basename of the asset resource.
      String get fileBasename {
        final basename = path.basename(assetName);
        return basename;
      }
    
      /// The no extension file basename of the asset resource.
      String get fileBasenameNoExtension {
        final basenameWithoutExtension = path.basenameWithoutExtension(assetName);
        return basenameWithoutExtension;
      }
    
      /// The file extension name of the asset resource.
      String get fileExtname {
        final extension = path.extension(assetName);
        return extension;
      }
    
      /// The directory path name of the asset resource.
      String get fileDirname {
        var dirname = assetName;
        if (packageName != null) {
          final packageStr = "packages/$packageName/";
          dirname = dirname.replaceAll(packageStr, "");
        }
        final filenameStr = "$fileBasename/";
        dirname = dirname.replaceAll(filenameStr, "");
        return dirname;
      }
    }
    ```

13. 遍历`non_svg_image_asset`数组，根据下面的模板生成`_R_Image_AssetResource`类，追加写入`r.g.dart`。

    - `#{asset_comment}`：资产资源变量的文档注释；其定义为`asset: #{asset_name}`
    - `#{escaped_asset_name}`：经过转义处理的`asset`；转义主要处理`$`字符；该值得生成可参考`xx`

    ```dart
    // ignore: camel_case_types
    class _R_Image_AssetResource {
      const _R_Image_AssetResource();
      
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      final #{asset_id} = const AssetResource("#{escaped_asset_name}", packageName: R.package);
    }
    ```

14. 遍历`svg_image_asset`数组，根据下面的模板生成`_R_Svg_AssetResource`类，追加写入`r.g.dart`。

    ```dart
    // ignore: camel_case_types
    class _R_Svg_AssetResource {
      const _R_Svg_AssetResource();
      
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      final #{asset_id} = const AssetResource("#{escaped_asset_name}", packageName: R.package);
    }
    ```

15. 遍历`text_asset`数组，根据下面的模板生成`_R_Text_AssetResource`类，追加写入`r.g.dart`。

    ```dart
    // ignore: camel_case_types
    class _R_Text_AssetResource {
      const _R_Text_AssetResource();
      
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      final #{asset_id} = const AssetResource("#{escaped_asset_name}", packageName: R.package);
    }
    ```

16. 遍历`non_svg_image_asset`数组，根据下面的模板生成`_R_Image`类，追加写入`r.g.dart`。

    ```dart
    /// This `_R_Image` class is generated and contains references to static non-svg type image asset resources.
    // ignore: camel_case_types
    class _R_Image {
      const _R_Image();
    
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      AssetImage #{asset_id}() {
        return AssetImage(asset.#{asset_id}.keyName);
      }
    }
    ```

17. 遍历`svg_image_asset`数组，根据下面的模板生成`_R_Svg`类，追加写入`r.g.dart`。

    ```dart
    /// This `_R_Svg` class is generated and contains references to static svg type image asset resources.
    // ignore: camel_case_types
    class _R_Svg {
      const _R_Svg();
    
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      AssetSvg #{asset_id}({@required double width, @required double height}) {
        final imageProvider = AssetSvg(asset.#{asset_id}.keyName, width: width, height: height);
        return imageProvider;
      }
    }
    ```

18. 遍历`text_asset`数组，根据下面的模板生成`_R_Text`类，追加写入`r.g.dart`。

    ```dart
    /// This `_R_Text` class is generated and contains references to static text asset resources.
    // ignore: camel_case_types
    class _R_Text {
      const _R_Text();
    
      /// #{asset_comment}
      // ignore: non_constant_identifier_names
      Future<String> #{asset_id}() {
        final str = rootBundle.loadString(asset.#{asset_id}.keyName);
        return str;
      }
    }
    ```

19. 遍历`font_family_config`数组，根据下面的模板生成`_R_Font_Family`类，追加写入`r.g.dart`。

    - `#{font_family_comment}`：字体家族变量的文档注释；其定义为`font family: #{font_family_name}`

    ```dart
    /// This `_R_FontFamily` class is generated and contains references to static font asset resources.
    // ignore: camel_case_types
    class _R_FontFamily {
      const _R_FontFamily();
      
      /// #{font_family_comment}
      // ignore: non_constant_identifier_names
      final #{font_family_id} = "#{font_family_name}";
    }
    ```

20. 结束操作，保存`r.g.dart`。

21. 调用flutter工具对`r.g.dart`进行格式化操作。

22. 调用flutter工具，为flutter工程获取依赖。

23. 判断警告日志数组是否为空，若不为空，输出所有警告日志。

#### start monitor

1. 进行环境检测；若发现不合法的环境，则抛出异常，终止当前进程：

   - 检测当前flutter工程根目录是否存在`pubspec.yaml`。

   - 检测当前`pubspec.yaml中`是否存在`flr_conig`。

   - 检测当前`flr_config`中的`resource_dir`配置是否合法：

      判断合法的标准是：`assets`配置或者`fonts`配置了至少1个`legal_resource_dir`。

2. 执行一次`flr generate`操作；

3. 获取`legal_resource_dir`数组：

   - 从`flr_config`中的`assets`配置获取`assets_legal_resource_dir`数组；
   - 从`flr_config`中的`fonts`配置获取`fonts_legal_resource_dir`数组；
   - 合并`assets_legal_resource_dir`数组和`fonts_legal_resource_dir`数组为`legal_resource_dir`数组。

4. 启动资源变化监控服务；
   - 启动一个文件监控服务，对`legal_resource_dir`数组中的资源目录进行文件监控
   - 若服务检测到资源变化（资源目录下的发生增/删/改文件），则执行一次`flr generate`操作

#### stop monitor

1. 停止资源变化监控服务


