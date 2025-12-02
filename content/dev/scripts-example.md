---
title: 一些sh脚本
date: 2025-10-25
tags:
  - shell
  - code
  - linux
ai: false
categories: handbook
---

# scripts example

```
find . -name "*.md" -type f -exec grep -lE "Finished" {} + | while read file; do
  if ! grep -q "2025-05-31" "$file"; then
    number=$(awk '/^pages: [0-9]+/ {print $2}' "$file")
    if [ -n "$number" ]; then
      sed -i "/Finished/i\\- 2025-05-31: $number" "$file"
      echo "已处理: $file - 插入: - 2025-05-31: $number"
    else
      echo "跳过: $file (未找到pages数字或有效日期)"
    fi
  else
    echo "排除: $file (包含 2025-05-31)"
  fi
done
```

```
# 创建备份和日志
backup_dir="backup_$(date +%Y%m%d)"
mkdir -p "$backup_dir"
log_file="replace_log.txt"

echo "开始替换操作 $(date)" > "$log_file"

find . -name "*.md" -type f | while read file; do
    # 检查文件是否包含目标字符串
    if grep -q "- 2025-06-01: Finished" "$file"; then
        # 创建备份
        cp "$file" "$backup_dir/$(basename "$file")"
        # 执行替换
        sed -i 's/- 2025-06-01: Finished/- 2025-05-31: Finished/g' "$file"
        # 记录日志
        echo "已修改: $file" >> "$log_file"
        # 显示变更
        echo "=== $file 修改内容 ==="
        diff "$backup_dir/$(basename "$file")" "$file" || true
    else
        echo "无修改: $file" >> "$log_file"
    fi
done

echo "操作完成，备份在 $backup_dir，日志见 $log_file"
```

```
find . -name "*.md" -type f -exec grep -l "lists: book" {} + | \
while read file; do
    if ! grep -q "Finished" "$file"; then
        # 确保文件末尾有空行（避免追加内容粘连）
        [ -n "$(tail -c 1 "$file")" ] && echo >> "$file"
        # 追加内容
        echo "- 2025-06-01: Finished" >> "$file"
        echo "已处理: $file"
    else
        echo "跳过: $file (已包含 Finished)"
    fi
done
```

```
find . -name "*.md" -type f -exec grep -L "Finished" {} + | while read file; do
    awk 'BEGIN {done=0} /^---/ && !done {print; print "- 2025-06-01: Finished"; done=1; next} 1' "$file" > tmp && mv tmp "$file"
done
```
