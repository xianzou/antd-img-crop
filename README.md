# antd-img-crop

An image cropper for Ant Design [Upload](https://ant.design/components/upload/).

[![npm](https://img.shields.io/npm/v/antd-img-crop.svg?style=flat-square)](https://www.npmjs.com/package/antd-img-crop)
[![npm](https://img.shields.io/npm/dt/antd-img-crop?style=flat-square)](https://www.npmtrends.com/antd-img-crop)
[![npm bundle size](https://img.shields.io/bundlephobia/minzip/antd-img-crop?style=flat-square)](https://bundlephobia.com/result?p=antd-img-crop)
[![GitHub](https://img.shields.io/github/license/nanxiaobei/antd-img-crop?style=flat-square)](https://github.com/nanxiaobei/antd-img-crop/blob/master/LICENSE)

English | [简体中文](./README.zh-CN.md)

## Demo

[Live demo](https://codesandbox.io/s/antd-img-crop-4qoom5p9x4)

## Install

```sh
yarn add antd-img-crop
```

## Usage

```jsx harmony
import ImgCrop from 'antd-img-crop';
import { Upload } from 'antd';

const Demo = () => (
  <ImgCrop>
    <Upload>+ Add image</Upload>
  </ImgCrop>
);
```

## Props

| Prop         | Type                 | Default        | Description                                                           |
| ------------ | -------------------- | -------------- | --------------------------------------------------------------------- |
| aspect       | `number`             | `1 / 1`        | Aspect of crop area , `width / height`                                |
| shape        | `string`             | `'rect'`       | Shape of crop area, `'rect'` or `'round'`                             |
| grid         | `boolean`            | `false`        | Show grid of crop area (third-lines)                                  |
| quality      | `number`             | `0.4`          | Image quality, `0 ~ 1`                                                |
| fillColor    | `string`             | `white`        | Fill color when cropped image smaller than canvas                     |
| zoom         | `boolean`            | `true`         | Enable zoom for image                                                 |
| rotate       | `boolean`            | `false`        | Enable rotate for image                                               |
| minZoom      | `number`             | `1`            | Minimum zoom factor                                                   |
| maxZoom      | `number`             | `3`            | Maximum zoom factor                                                   |
| modalTitle   | `string`             | `'Edit image'` | Title of modal                                                        |
| modalWidth   | `number` \| `string` | `520`          | Width of modal in pixels number or percentages                        |
| modalOk      | `string`             | `'OK'`         | Text of confirm button of modal                                       |
| modalCancel  | `string`             | `'Cancel'`     | Text of cancel button of modal                                        |
| beforeCrop   | `function`           | -              | Call before modal open, if return `false`, it'll not open             |
| cropperProps | `object`             | -              | Props of [react-easy-crop] (\* [existing props] cannot be overridden) |

## Styles

To prevent overwriting the custom styles to `antd`, `antd-img-crop` does not import the style files of components.

Therefore, if your project configured `babel-plugin-import`, and not use `Modal` or `Slider`, you need to import the styles yourself:

```js
import 'antd/es/modal/style';
import 'antd/es/slider/style';
```

## License

[MIT License](https://github.com/nanxiaobei/antd-img-crop/blob/master/LICENSE) (c) [nanxiaobei](https://mrlee.me/)

[react-easy-crop]: https://github.com/ricardo-ch/react-easy-crop#props
[existing props]: https://github.com/nanxiaobei/antd-img-crop/blob/master/src/index.jsx#L67-L83


示例
```js
import styles from './ImageUpload.scss';
import React, { useState, Fragment } from 'react';
import { Toast, Preview, Modal } from '@mshare/mshareui';
import { getContextPath } from '@/utils';
import ImgCrop from 'antd-img-crop';
import Upload from 'rc-upload';
// import upLoader from '@/assets/images/up-loader.png';
import womanTop from '@/assets/images/woman-top.png';
import manTop from '@/assets/images/man-top.png';
import imgAddNew from '@/assets/images/img-add-new.png';
import 'antd/es/modal/style/index.css';
import 'antd/es/slider/style/index.css';

const errorModal = msg => {
    Modal.alert('提示', msg);
};

export default function (props) {
    const { value, onChange, url, studentInfo = {} } = props || {};
    const [isOpen, setIsOpen] = useState(false);
    const contextPath = getContextPath();

    const result = ({ fileId }) => ({
        src: `${contextPath}api/file/file/?fileId=${fileId}`,
        thumbnail: `${contextPath}api/file/file?fileId = ${fileId} `,
        w: 0,
        h: 0,
    });
    const uploadProps = {
        action: url,
        onStart: () => {
            Toast.loading('上传中...', 0);
        },
        onSuccess: response => {
            Toast.hide();
            const { data, status, message } = response;
            const [files] = data;

            if (status === '1200') {
                const _value = (value || []).concat(files);

                onChange(_value);
            } else {
                errorModal(message || '上传失败');
            }

        },
        onError(err) {
            errorModal(err);
        },
        accept: 'image/*',
        style: { display: 'inline-block' },
    };

    const items = (value || []).map(item => result(item));

    let defaultImg = manTop;

    if (studentInfo.xb) {
        defaultImg = studentInfo.xb === '女' ? womanTop : manTop;
    }
    return (
        <Fragment>
            <div className={styles.ImageUpload}>
                {
                    items.map((item, index) => (
                        <div className={styles.thumbnailImg} key={index}>
                            <img src={item.src} onClick={() => setIsOpen(!isOpen)} alt="" />
                            <div
                                className={styles.deleteImg}
                                onClick={() => {
                                    const temp = value.filter(row => row.fileId === item.fileId);

                                    onChange(temp);
                                }}
                            />
                        </div>
                    ))
                }
                {
                    items && items.length < 1 && (
                        <ImgCrop
                            grid
                            modalTitle="编辑照片"
                            modalOk="确定"
                            modalCancel="取消"
                        >
                            <Upload {...uploadProps}>
                                <div className={styles.imgAddBtn} >
                                    <img src={defaultImg} alt="" />
                                    <img src={imgAddNew} className={styles.addNew} alt="" />
                                </div>
                            </Upload>
                        </ImgCrop>
                    )
                }

                <Preview isOpen={isOpen} items={items} onClose={() => setIsOpen(!isOpen)} />
            </div>
        </Fragment>
    );
}

```
使用
```js
  <ImageUploadTest
      url={`${contextPath}api/file/upload`}
      value={value}
      onChange={e => onImgUploadChange(e)}
      studentInfo={studentInfo}
  />
```
