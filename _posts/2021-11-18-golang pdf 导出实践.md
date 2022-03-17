---
layout: post
title:  "golang pdf 导出实践"
date:   2021-11-18
description: 'golang pdf 导出实践'
category: notes

---

Maroto 可以类似与 bootstrap 这样的前端库一样对 pdf 页面布局排版，使用行、列和组件来构建 pdf 文档，也可以支持图片、二维码等

在项目目录下，引入依赖：

```bash
go get -u github.com/johnfercher/maroto
go get github.com/johnfercher/maroto/internal@v0.31.0
go get github.com/brianvoe/gofakeit/v6
```


示例代码：

`main.go`

```golang
package main

import (
	"fmt"
	"os"

	"github.com/johnfercher/maroto/pkg/color"
	"github.com/johnfercher/maroto/pkg/consts"
	"github.com/johnfercher/maroto/pkg/pdf"
	"github.com/johnfercher/maroto/pkg/props"

	"github.com/divrhino/fruitful-pdf/data"
)

func main() {
	m := pdf.NewMaroto(consts.Portrait, consts.A4)
	m.AddUTF8Font("NotoSansSC", "", "./fonts/NotoSansSC-Regular.ttf")
	m.AddUTF8Font("NotoSansSC", "I", "./fonts/NotoSansSC-Regular.ttf")
	m.AddUTF8Font("NotoSansSC", "B", "./fonts/NotoSansSC-Regular.ttf")
	m.AddUTF8Font("NotoSansSC", "BI", "./fonts/NotoSansSC-Regular.ttf")
	m.SetPageMargins(20, 10, 20)

	buildHeading(m)
	buildGroupHeader(m, "基础信息")
	buildBaseInfo(m)
	buildGroupHeader(m, "土地信息")
	buildLandInfo(m)
	buildGroupHeader(m, "宗地信息")
	buildFruitList(m)
	buildGroupHeader(m, "土地规划")
	buildImage(m, "images/001.png")

	err := m.OutputFileAndClose("pdfs/001.pdf")
	if err != nil {
		fmt.Println("⚠️  Could not save PDF:", err)
		os.Exit(1)
	}

	fmt.Println("PDF saved successfully")
}

func buildHeading(m pdf.Maroto) {
	m.RegisterHeader(func() {
		m.Row(20, func() {
			m.Col(12, func() {
				err := m.FileImage("images/logo.png", props.Rect{
					Center:  true,
					Percent: 55,
				})

				if err != nil {
					fmt.Println("Image file was not loaded 😱 - ", err)
				}
			})
		})
		m.Row(10, func() {
			m.Col(12, func() {
				m.Text("小黄鞋大宗物业交易平台", props.Text{
					Size:   12.0,
					Top:    3.0,
					Family: "NotoSansSC",
					Style:  consts.BoldItalic,
					Align:  consts.Center,
					Color:  getDarkPurpleColor(),
				})
			})
		})
	})
}

// 头部
func buildGroupHeader(m pdf.Maroto, name string) {
	m.Row(10, func() {
		m.Col(1, func() {
			m.Text(name, props.Text{
				Top:    5,
				Family: "NotoSansSC",
				Style:  consts.BoldItalic,
				Align:  consts.Left,
				Color:  getTealColor(),
			})
		})
	})
	m.Line(1.0)
}

func buildBaseInfo(m pdf.Maroto) {

	m.Row(10, func() {
		buildFormItem(m, "城区：", "宝安区")
		buildFormItem(m, "片区：", "宝安中心")
	})

	m.Row(10, func() {
		buildFormLine(m, "地址：", "西安市周至县振兴北路金凤幼,儿园西北侧约150米")
	})
}

func buildLandInfo(m pdf.Maroto) {
	m.Row(10, func() {
		buildFormItem(m, "土地来源：", "深圳经济发展公司签订租赁")
		buildFormItem(m, "土地性质：", "住宅-R0")
	})
	m.Row(10, func() {
		buildFormLine(m, "土地使用年限：", "50 年，2001 年 01 月 24 日 - 2032 年 03 月 20 日")
	})
	m.Row(10, func() {
		buildFormLine(m, "土地用途：", "厂房出租")
	})
}

// 列
func buildFormItem(m pdf.Maroto, label, text string) {
	m.Col(2, func() {
		m.Text(label, props.Text{
			// Size:        10.0,
			Style:       consts.BoldItalic,
			Family:      "NotoSansSC",
			Align:       consts.Left,
			Top:         3.0,
			Extrapolate: false,
			Color: color.Color{
				Red:   10,
				Green: 20,
				Blue:  30,
			},
		})
	})
	m.Col(4, func() {
		m.Text(text, props.Text{
			Top:    3,
			Family: "NotoSansSC",
			Align:  consts.Left,
			Color:  getDarkPurpleColor(),
		})
	})
}

// 整行
func buildFormLine(m pdf.Maroto, label, text string) {
	m.Col(2, func() {
		m.Text(label, props.Text{
			// Size:        10.0,
			Style:       consts.BoldItalic,
			Family:      "NotoSansSC",
			Align:       consts.Left,
			Top:         3.0,
			Extrapolate: false,
			Color: color.Color{
				Red:   10,
				Green: 20,
				Blue:  30,
			},
		})
	})
	m.Col(10, func() {
		m.Text(text, props.Text{
			Family:          "NotoSansSC",
			Align:           consts.Left,
			Top:             3,
			Extrapolate:     false,
			VerticalPadding: 1.0,
			Color:           getDarkPurpleColor(),
		})
	})
}

func buildFruitList(m pdf.Maroto) {
	tableHeadings := []string{"宗地号", "占地面积", "有证占地面积", "无证占地面积", "建筑面积", "有证建筑面积", "无证建筑面积", "建筑现状"}
	contents := data.FruitList(5)
	lightPurpleColor := getLightPurpleColor()

	m.SetBackgroundColor(color.NewWhite())
	m.TableList(tableHeadings, contents, props.TableList{
		HeaderProp: props.TableListContent{
			Size:      8,
			Family:    "NotoSansSC",
			GridSizes: []uint{1, 1, 2, 2, 1, 2, 2, 1},
		},
		ContentProp: props.TableListContent{
			Size:      7,
			Family:    "NotoSansSC",
			GridSizes: []uint{1, 1, 2, 2, 1, 2, 2, 1},
		},
		Align:                consts.Center,
		AlternatedBackground: &lightPurpleColor,
		HeaderContentSpace:   1,
		Line:                 false,
	})

}

func getDarkPurpleColor() color.Color {
	return color.Color{
		Red:   88,
		Green: 80,
		Blue:  99,
	}
}

func getLightPurpleColor() color.Color {
	return color.Color{
		Red:   210,
		Green: 200,
		Blue:  230,
	}
}

func getTealColor() color.Color {
	return color.Color{
		Red:   3,
		Green: 166,
		Blue:  166,
	}
}

func buildImage(m pdf.Maroto, src string) {
	m.Row(100, func() {
		m.Col(12, func() {
			err := m.FileImage(src, props.Rect{
				Center: false,
				// Percent: 100,
				Top: 3,
			})

			if err != nil {
				fmt.Println("Image file was not loaded 😱 - ", err)
			}
		})
	})

}
```

导出的展示：

<img src="/assets/images/blog/maroto-example.png" />