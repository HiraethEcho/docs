---
title: work with ai on a board
summary: Another way work with ai
---

# Board

## Simple Motivation

Another way work with ai.

the center is files (pinned on a board), human and llm modify the files by diff. the files have higher priority in context.
the essential way for human to commutate with llm is the diff file.  
(say there is main branch and a dev branch, we work on dev branch. then there are two branches `human` and `ai` rebased on dev, human writes on human branch. there should be a orchestrator that give llm the diff file on human branch, and merge, rebase these two branches on dev branch. maybe `jj` is better on this workflow?)

there can be another channel where human and llm chat with each other, human can talk to llm with "pointing some context in the files", but not the llm which write files.

The idea is, there should be a llm that "do the actual work", and it get the information in a more efficient way, not using the chat history.

the human attention is poor, and we have to through away a lot stuffs that we consider not important, or dead end.

## context

上下文应该是有不同层级的，它们在 llm 中地位应该不同。
Work Space 中的代码/文本应当是一等公民，由某个特化的 llm 来处理，并且应该是**无历史**的。
（人，或其他 llm/agent）对文件的修改也应该作为一个整体，同样被一个特化的 llm 处理，通过 diff 来理解 _修改_。显然人类的修改应该地位更高，对文本（workspace里那些东西）的修改，应该和在对话框里的 prompt 是同样地位的。
此外，还有一个传统对话的 llm，用来保留全部记忆。但它不直接参与修改，而只是负责统筹。

> vibe coding 实践中的习惯：如果在agent操作时人类干预直接修改文件，这很难被 llm 感知。有时候简单改一些 typo 或小细节，为了嵌入到模型记忆中，需要通过 prompt 实现（还容易漂移，非常麻烦）

学习一下 Aider 的思路。用 jujustu 来管理 diff.

## 哲学

从哲学理论中吸收一些（通常不科学的）概念。
在后现代语境下，什么是 **文本/context** ？生活材料本身也是文本，概念的解释只能在 context 中。
从语言哲学里学习一下，怎样划分原子语句。

## A demo

一个非常初级的实现 demo 是 [pi-board 插件](/projects/pi-board)，仅仅实现了引用文件文本和对话历史的部分。
