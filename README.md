# PDF Reader

A simple Go library which enables reading PDF files. Forked from https://github.com/rsc/pdf

Features
  - Get plain text content (without format)
  - Get Content (including all font and formatting information)

## Install:

`go get -u github.com/ledongthuc/pdf`


## Read plain text

```golang
package main

import (
	"bytes"
	"fmt"

	"github.com/ledongthuc/pdf"
)

func main() {
	content, err := readPdf("test.pdf") // Read local pdf file
	if err != nil {
		panic(err)
	}
	fmt.Println(content)
	return
}

func readPdf(path string) (string, error) {
	f, r, err := pdf.Open(path)
	// remember close file
    defer f.Close()
	if err != nil {
		return "", err
	}
	var buf bytes.Buffer
    b, err := r.GetPlainText()
    if err != nil {
        return "", err
    }
    buf.ReadFrom(b)
	return buf.String(), nil
}
```

## Read all text with styles from PDF

```golang
func readPdf2(path string) (string, error) {
	f, r, err := pdf.Open(path)
	// remember close file
	defer f.Close()
	if err != nil {
		return "", err
	}
	totalPage := r.NumPage()

	for pageIndex := 1; pageIndex <= totalPage; pageIndex++ {
		p := r.Page(pageIndex)
		if p.V.IsNull() {
			continue
		}
		var lastTextStyle pdf.Text

		texts := p.Content().Text
		for i, text := range texts {
			if isSameSentence(text, lastTextStyle) {
				lastTextStyle.S = lastTextStyle.S + text.S
				if i < len(texts)-1 {
					continue
				}
			}

			fmt.Printf("Font: %s, Font-size: %f, x: %f, y: %f, content: %s \n", lastTextStyle.Font, lastTextStyle.FontSize, lastTextStyle.X, lastTextStyle.Y, lastTextStyle.S)
			lastTextStyle = text
		}
	}
	return "", nil
}

func isSameSentence(t1, t2 pdf.Text) bool {
	return t1.Y == t2.Y
}
```

## Demo
![Run example](https://i.gyazo.com/01fbc539e9872593e0ff6bac7e954e6d.gif)
