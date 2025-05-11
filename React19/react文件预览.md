## pdf 预览

可以用浏览器自带预览功能解决

```js
const blob = await fetch(url).then((res) => res.blob());
const blobUrl = URL.createObjectURL(blob);
window.open(blobUrl, "_blank");
```

将云端地址转流并创建一个本地临时 url，用浏览器打开这个 url 就能预览

## xls/xlsx 预览

```bush
cnpm install xlsx file-saver
```

以下是一个使用 xlsx 实现的简单表格渲染案例

```js
const fetchAndPreviewExcel = async (url: string) => {
  const response = await fetch(url);
  //   云端下载文件转二进制
  const arrayBuffer = await response.arrayBuffer();
  //   传入二进制数据给xlsx组件
  const workbook = XLSX.read(arrayBuffer, { type: "array" });
  //手动构建二维数据
  function getFullSheetData(worksheet: XLSX.WorkSheet): any[][] {
    // 拿到表格起止行数据
    const range = XLSX.utils.decode_range(worksheet["!ref"] || "A1");
    const rows: any[][] = [];
    // 循环构建表格数据
    for (let row = range.s.r; row <= range.e.r; row++) {
      const rowData: any[] = [];
      for (let col = range.s.c; col <= range.e.c; col++) {
        const cellAddress = XLSX.utils.encode_cell({ r: row, c: col });
        const cell = worksheet[cellAddress];
        rowData.push(cell ? cell.v : "");
      }
      rows.push(rowData);
    }

    return rows;
  }
  // 使用antd的tabs构建工作簿
  const tabItems: TabsProps["items"] = workbook.SheetNames.map(
    (sheetName, sheetIndex) => {
      const worksheet = workbook.Sheets[sheetName];
      const jsonData = getFullSheetData(worksheet); //用替代函数

      const maxColumns = Math.max(...jsonData.map((row) => row.length));

      const tableColumns = Array.from({ length: maxColumns }).map(
        (_, index) => ({
          title: XLSX.utils.encode_col(index), // 列表头
          dataIndex: `col_${index}`,
          key: `col_${index}`,
        })
      );

      const tableDataSource = jsonData.map((row, rowIndex) => {
        const rowData: any = { key: rowIndex };
        for (let i = 0; i < maxColumns; i++) {
          rowData[`col_${i}`] = row[i] ?? "";
        }
        return rowData;
      });
      // 返回每个工作簿对应的表格数据
      return {
        label: sheetName,
        key: `sheet-${sheetIndex}`,
        children: (
          <Table
            columns={tableColumns}
            dataSource={tableDataSource}
            pagination={false}
            bordered
            scroll={{ x: 1200, y: 600 }}
          />
        ),
      };
    }
  );

  setViewerDom(<Tabs items={tabItems} />);
};
```

## docx 预览

```bush
cnpm install docx-preview
```

这个插件需要先建立一个容器用来展示处理好的 docx 文件

```js
<div id={id} style={{ maxHeight: "600px", overflowY: "scroll" }}></div>
```

再将容器存入 panel 中，将云端 url 转为二进制，通过 docx.renderAsync（）将二进制文件渲染到容器中

```js
const panel = document.getElementById(id);
// getFileUrl为文件转二进制操作
const blob = await getFileUrl(url);
await docx.renderAsync(blob, panel);
```

```js
const getFileUrl = async (url: string) => {
  const data = await fetch(url);
  const blob = await data.blob();
  return blob;
};
```

以上就是目前市面常用的文件预览方式
