---
title: easyExcel的使用及合并单元格的处理
catalog: true
date: 2020-09-24 09:56:10
subtitle: excel的处理
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
## easyexcel的使用及合并单元格的处理
#### 一.介绍

```javascript
[github地址](https://github.com/alibaba/easyexcel)
easyexcel是阿里巴巴在poi基础上封装的一款专门处理excel文件的工具jar。
java解析、生成Excel比较有名的框架有Apache poi、jxl。
但他们都存在一个严重的问题就是非常的耗内存，
poi有一套SAX模式的API可以一定程度的解决一些内存溢出的问题，
但POI还是有一些缺陷，比如07版Excel解压缩以及解压后存储都是在内存中完成的，
内存消耗依然很大。easyexcel重写了poi对07版Excel的解析，
能够原本一个3M的excel用POI sax依然需要100M左右内存降低到几M，
并且再大的excel不会出现内存溢出，03版依赖POI的sax模式。
在上层做了模型转换的封装，让使用者更加简单方便。
```
#### 二.easyexcel合并单元格的使用

```javascript
easyexcel的简单使用这里就不多做介绍了，github项目里的单元测试有很多例子
[easyexcel项目地址](https://github.com/alibaba/easyexcel)
这里只介绍高级用法——合并单元格的导入和导出。注意：这里的合并单元格指表内容合并，
导出时表头的合并可直接在@ExcelProperty里面处理，这里不多做介绍。
```

##### 1.合并单元格的导入

```javascript
读取合并单元格时会出现一个问题，合并的单元格读取数据时只有第一个会读到数据，
其他的数据为null，这种怎么处理呢？上代码：
```

```javascript
public class ExcelTest extends BaseTest {
    @Test
    public void test1() {
    	String filePath = "D:/test.xlsx";
    	String headRowNumber = 1;//表头行数
    	//StudentListener里就是处理读取合并单元格的逻辑
		List<StudentDO> dataList = EasyExcel.read(new File(filePath ), StudentDO.class,
                    new StudentListener()).sheet().headRowNumber(headRowNumber ).doReadSync();
    }
}
```

```javascript
public class StudentListener extends AnalysisEventListener<StudentDO> {
	//上一行数据
    private StudentDO last;

    /**
     * 合并单元格读取处理
     * 当前数据为null则取上一行的值
     */
    @Override
    public void invoke(StudentDO studentDO , AnalysisContext analysisContext) {
        String name = studentDO .getName();
        String addr = studentDO.getAddr();
        if(StringUtils.isBlank(name)){
            studentDO .setName(last.getName());
        }
        if(StringUtils.isBlank(addr)){
            studentDO.setAddr(last.getAddr());
        }
        last = studentDO;
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {


    }
```
##### 2.导出合并单元格

```javascript
List<StudentDO > studentDOList = studenMapper.queryStudentList();
String fileName = System.currentTimeMillis() + ".xlsx";
HttpServletResponse response = SpringMvcUtils.getHttpResponse();
response.setHeader("Content-disposition", "attachment; filename=" + fileName);
response.setCharacterEncoding("UTF-8");
//需要合并的索引列
int[] mergeColIndex = {1, 2, 3, 4};
//需要从第一行开始，列头第一行
int mergeRowIndex = 1;
//头样式
WriteCellStyle headWriteCellStyle = new WriteCellStyle();
WriteFont headWriteFont = new WriteFont();
//字体大小
headWriteFont.setFontHeightInPoints((short) 9);
//加粗
headWriteFont.setBold(false);
headWriteCellStyle.setWriteFont(headWriteFont);
//单元格上下居中
headWriteCellStyle.setVerticalAlignment(VerticalAlignment.CENTER);
//内容样式
WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
WriteFont contentWriteFont = new WriteFont();
contentWriteFont.setFontHeightInPoints((short) 9);
//设置字体样式
contentWriteFont.setFontName("Arial");
contentWriteCellStyle.setWriteFont(contentWriteFont);
contentWriteCellStyle.setVerticalAlignment(VerticalAlignment.CENTER);
//表头和表格内容样式策略
HorizontalCellStyleStrategy horizontalCellStyleStrategy =
new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);
EasyExcel.write(response.getOutputStream(), McdDataParseDTO.class)
                .excelType(ExcelTypeEnum.XLSX).autoCloseStream(true)
                .registerWriteHandler(new CellMergeStrategy(mergeColIndex, mergeRowIndex))
                .registerWriteHandler(horizontalCellStyleStrategy)
                .sheet("学生表").doWrite(dataList);        
```

```javascript
**
 * excel导出数据内容单元格合并规则
 *
 * @Author shenghao.xiao
 * @Date 2020/9/17
 **/
public class CellMergeStrategy implements CellWriteHandler {
    /**
     * 合并列的范围索引
     */
    private int[] mergeColumnIndex;
    /**
     * 合并起始行索引
     */
    private int mergeRowIndex;

    public CellMergeStrategy(int[] mergeColumnIndex, int mergeRowIndex) {
        this.mergeColumnIndex = mergeColumnIndex;
        this.mergeRowIndex = mergeRowIndex;
    }

    @Override
    public void beforeCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Row row, Head head, Integer integer, Integer integer1, Boolean aBoolean) {

    }

    @Override
    public void afterCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Cell cell, Head head, Integer integer, Boolean aBoolean) {

    }

    @Override
    public void afterCellDataConverted(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, CellData cellData, Cell cell, Head head, Integer integer, Boolean aBoolean) {

    }

    @Override
    public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, List<CellData> list, Cell cell, Head head, Integer integer, Boolean aBoolean) {
        //当前行
        int curRowIndex = cell.getRowIndex();
        //当前列
        int curColIndex = cell.getColumnIndex();
        if (curColIndex >= mergeRowIndex) {
            for (int columnIndex : mergeColumnIndex) {
                if (curColIndex == columnIndex) {
                    mergeWithPrevRow(writeSheetHolder, cell, curRowIndex, curColIndex);
                    break;
                }
            }
        }
    }

    private void mergeWithPrevRow(WriteSheetHolder writeSheetHolder, Cell cell, int curRowIndex, int curColIndex) {
        //第一行数据不用合并
        if (curRowIndex == 0) {
            return;
        }
        //获取当前行的当前列的数据和上一行的当前列数据，通过上一行数据是否相同进行合并
        Object curData = cell.getCellTypeEnum() == CellType.STRING ? cell.getStringCellValue() : cell.getNumericCellValue();
        Cell preCell = cell.getSheet().getRow(curRowIndex - 1).getCell(curColIndex);
        Object preData = preCell.getCellTypeEnum() == CellType.STRING ? preCell.getStringCellValue() : preCell.getNumericCellValue();

        //比较当前行的第一列的单元格与上一行是否相同，相同合并当前单元格与上一行
        if (curData.equals(preData)) {
            Sheet sheet = writeSheetHolder.getSheet();
            List<CellRangeAddress> mergedRegions = sheet.getMergedRegions();
            boolean isMerged = false;
            for (int i = 0; i < mergedRegions.size() && !isMerged; i++) {
                CellRangeAddress cellAddresses = mergedRegions.get(i);
                //若上 一个单元格已经被合并，则先移出原有的合并单元，再重新添加合并单元
                if (cellAddresses.isInRange(curRowIndex - 1, curColIndex)) {
                    sheet.removeMergedRegion(i);
                    cellAddresses.setLastRow(curRowIndex);
                    sheet.addMergedRegion(cellAddresses);
                    isMerged = true;
                }
            }
            //若上一个单元格未被合并，则新增合并单元
            if (!isMerged) {
                CellRangeAddress cellAddresses = new CellRangeAddress(curRowIndex - 1, curRowIndex, curColIndex, curColIndex);
                sheet.addMergedRegion(cellAddresses);
            }
        }
    }
}
```

