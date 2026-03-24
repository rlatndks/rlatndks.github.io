---
layout: post
title: "선택한 오브젝트 객체 수 확인"
date: 2026-03-24
categories: ["maya"]
tags: ["#object#number#count"]
---

$selected = `ls -sl`;
$counter = 0;
for($count in $selected)
{
    $counter = $counter +1;
}
print $counter;