---
title: 加密算法
date: 2025-10-17
tags:
  - tech
categories: 规范协议技术
topics:
draft: true
---

# 加密算法

记录一点简单的加密方法。

## 1

pub(7,33) pri(?,33)

\[
  x^7 \equiv y \mathrm{mod} 33 
\]
and
\[
  y^3 \equiv x \mathrm{mod} 33 
\]

in fact, take primes p,q, and n=pq, T=(p-1)(q-1) and 1<e<T and d such that
\[
  de \equiv 1\mathrm{T}
\]

Then 
\[
  x^e \equiv y \mathrm{mod} n
\]
and
\[
  y^d \equiv x \mathrm{mod} n 
\]

## DH

prime P, generator G, a,b for alice and bob

if

\[
  G^a \equiv x \mathrm{mod} P
\]
and
\[
  G^b \equiv y \mathrm{mod} P
\]

then
\[
  x^b \equiv y^a \mathrm{mod} P
\]

## ECC, RSA
