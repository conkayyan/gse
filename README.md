# gse

Go efficient multilingual NLP and text segmentation; support english, chinese, japanese and other. 
And supports with [elasticsearch](https://github.com/vcaesar/go-gse-elastic) and [bleve](https://github.com/vcaesar/gse-bleve).

<!--<img align="right" src="https://raw.githubusercontent.com/go-ego/ego/master/logo.jpg">-->
<!--<a href="https://circleci.com/gh/go-ego/ego/tree/dev"><img src="https://img.shields.io/circleci/project/go-ego/ego/dev.svg" alt="Build Status"></a>-->

[![Build Status](https://github.com/go-ego/gse/workflows/Go/badge.svg)](https://github.com/go-ego/gse/commits/master)
[![CircleCI Status](https://circleci.com/gh/go-ego/gse.svg?style=shield)](https://circleci.com/gh/go-ego/gse)
[![codecov](https://codecov.io/gh/go-ego/gse/branch/master/graph/badge.svg)](https://codecov.io/gh/go-ego/gse)
[![Build Status](https://travis-ci.org/go-ego/gse.svg)](https://travis-ci.org/go-ego/gse)
[![Go Report Card](https://goreportcard.com/badge/github.com/go-ego/gse)](https://goreportcard.com/report/github.com/go-ego/gse)
[![GoDoc](https://godoc.org/github.com/go-ego/gse?status.svg)](https://godoc.org/github.com/go-ego/gse)
[![GitHub release](https://img.shields.io/github/release/go-ego/gse.svg)](https://github.com/go-ego/gse/releases/latest)
[![Join the chat at https://gitter.im/go-ego/ego](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/go-ego/ego?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<!-- [![Release](https://github-release-version.herokuapp.com/github/go-ego/gse/release.svg?style=flat)](https://github.com/go-ego/gse/releases/latest) -->
<!--<a href="https://github.com/go-ego/ego/releases"><img src="https://img.shields.io/badge/%20version%20-%206.0.0%20-blue.svg?style=flat-square" alt="Releases"></a>-->

[简体中文](https://github.com/go-ego/gse/blob/master/README_zh.md)

Gse is implements jieba by golang, and try add NLP support and more feature

## Feature:
- Support common, search engine, full mode, precise mode and HMM mode multiple word segmentation modes; 
- Support user and embed dictionary, Part-of-speech/POS tagging, analyze segment info, stop and trim words
- Support multilingual: English, Chinese, Japanese and other
- Support traditional chinese
- Support HMM cut text use Viterbi algorithm
- Support NLP by TensorFlow (in work)
- Named Entity Recognition (in work) 
- Supports with [elasticsearch](https://github.com/vcaesar/go-gse-elastic) and bleve
- run<a href="https://github.com/go-ego/gse/blob/master/tools/server/server.go"> JSON RPC service</a>.

## Algorithm:
- [Dictionary](https://github.com/go-ego/gse/blob/master/dictionary.go) with double array trie (Double-Array Trie) to achieve
- [Segmenter](https://github.com/go-ego/gse/blob/master/segmenter.go) algorithm is the shortest path (based on word frequency and dynamic programming), and DAG and HMM algorithm word segmentation.

## Text Segmentation speed:
- <a href="https://github.com/go-ego/gse/blob/master/tools/benchmark/benchmark.go"> single thread</a> 9.2MB/s
- <a href="https://github.com/go-ego/gse/blob/master/tools/benchmark/goroutines/goroutines.go">goroutines concurrent</a> 26.8MB/s. 
- HMM text segmentation single thread 3.2MB/s. (2core 4threads Macbook Pro).

## Binding:

[gse-bind](https://github.com/vcaesar/gse-bind), binding JavaScript and other, support more language.

## Install / update

```
go get -u github.com/go-ego/gse
```

## Use

```go
package main

import (
	"fmt"
	"regexp"

	"github.com/go-ego/gse"
	"github.com/go-ego/gse/hmm/pos"
)

var (
	text = "Hello world, Helloworld. Winter is coming! 你好世界."

	new, _ = gse.New("zh,testdata/test_dict3.txt", "alpha")

	seg gse.Segmenter
	posSeg pos.Segmenter
)

func main() {
	// Loading the default dictionary
	seg.LoadDict()
	// Loading the default dictionary with embed
	// seg.LoadDictEmbed()
	// 
	// Loading the simple chinese dictionary
	// seg.LoadDict("zh_s")
	// seg.LoadDictEmbed("zh_s")
	//
	// Loading the traditional chinese dictionary
	// seg.LoadDict("zh_t")
	// 
	// Loading the japanese dictionary
	// seg.LoadDict("jp")
	// 
	// Load the dictionary
	// seg.LoadDict("your gopath"+"/src/github.com/go-ego/gse/data/dict/dictionary.txt")

	cut()

	segCut()
}

func cut() {
	hmm := new.Cut(text, true)
	fmt.Println("cut use hmm: ", hmm)

	hmm = new.CutSearch(text, true)
	fmt.Println("cut search use hmm: ", hmm)
	fmt.Println("analyze: ", new.Analyze(hmm, text))

	hmm = new.CutAll(text)
	fmt.Println("cut all: ", hmm)

	reg := regexp.MustCompile(`(\d+年|\d+月|\d+日|[\p{Latin}]+|[\p{Hangul}]+|\d+\.\d+|[a-zA-Z0-9]+)`)
	text1 := `헬로월드 헬로 서울, 2021年09月10日, 3.14`
	hmm = seg.CutDAG(text1, reg)
	fmt.Println("Cut with hmm and regexp: ", hmm, hmm[0], hmm[6])
}

func analyzeAndTrim(cut []string) {
	a := seg.Analyze(cut, "")
	fmt.Println("analyze the segment: ", a)

	cut = seg.Trim(cut)
	fmt.Println("cut all: ", cut)

	fmt.Println(seg.String(text, true))
	fmt.Println(seg.Slice(text, true))
}

func cutPos() {
	po := seg.Pos(text, true)
	fmt.Println("pos: ", po)
	po = seg.TrimPos(po)
	fmt.Println("trim pos: ", po)

	pos.WithGse(seg)
	po = posSeg.Cut(text, true)
	fmt.Println("pos: ", po)

	po = posSeg.TrimWithPos(po, "zg")
	fmt.Println("trim pos: ", po)
}

func segCut() {
	// Text Segmentation
	tb := []byte(text)
	fmt.Println(seg.String(text, true))

	segments := seg.Segment(tb)
	// Handle word segmentation results, search mode
	fmt.Println(gse.ToString(segments, true))
}

```

[Look at an custom dictionary example](/examples/dict/main.go)

```Go
package main

import (
	"fmt"
	_ "embed"

	"github.com/go-ego/gse"
)

//go:embed test_dict3.txt
var testDict string

func main() {
	// var seg gse.Segmenter
	// seg.LoadDict("zh, testdata/test_dict.txt, testdata/test_dict1.txt")
	// seg.LoadStop()
	seg, err := gse.NewEmbed("zh, word 20 n"+testDict, "en")
	// seg.LoadDictEmbed()
	seg.LoadStopEmbed()

	text1 := "你好世界, Hello world"
	fmt.Println(seg.Cut(text1, true))
	fmt.Println(seg.String(text1, true))

	segments := seg.Segment([]byte(text1))
	fmt.Println(gse.ToString(segments))
}
```

[Look at an Chinese example](/examples/main.go)

[Look at an Japanese example](/examples/jp/main.go)

## Elasticsearch
How to use it with elasticsearch?

[go-gse-elastic](https://github.com/vcaesar/go-gse-elastic)

## Authors

- [Maintainers](https://github.com/orgs/go-ego/people)
- [Contributors](https://github.com/go-ego/gse/graphs/contributors)

## License

Gse is primarily distributed under the terms of "both the MIT license and the Apache License (Version 2.0)". 
See [LICENSE-APACHE](http://www.apache.org/licenses/LICENSE-2.0), [LICENSE-MIT](https://github.com/go-vgo/robotgo/blob/master/LICENSE).

Thanks for [sego](https://github.com/huichen/sego) and [jieba](https://github.com/fxsjy/jieba)([jiebago](https://github.com/wangbin/jiebago)).
