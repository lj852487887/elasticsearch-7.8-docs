# 同义词标记过滤器


`synonym`标记过滤器允许在分词过程中轻松处理同义词。同义词是使用配置文件配置的。以下是一个例子：

```json
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "whitespace",
            "filter": [ "synonym" ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "synonyms_path": "analysis/synonym.txt"
          }
        }
      }
    }
  }
}
```

上面配置了一个`synonym`过滤器，其路径为`analysis/synonym.txt`（相对于`config`位置）。然后使用过滤器配置`synonym`分词器。

此过滤器标记化链中出现在其前面的任何标记器和标记过滤器的同义词。

其他设置包括：

* `expand`（默认为`true`）。
* `lenient`（默认为`false`）。如果为 `true` 在解析同义词配置时忽略异常。需要注意的是，只有那些无法解析的同义词规则才会被忽略。例如，考虑以下要求：

```json
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "standard",
            "filter": [ "my_stop", "synonym" ]
          }
        },
        "filter": {
          "my_stop": {
            "type": "stop",
            "stopwords": [ "bar" ]
          },
          "synonym": {
            "type": "synonym",
            "lenient": true,
            "synonyms": [ "foo, bar => baz" ]
          }
        }
      }
    }
  }
}
```

对于上述请求，将跳过单词`bar`，但仍添加了映射`foo=>baz`。但是，如果添加的映射是`foo，baz=>bar`，则不会向同义词列表中添加任何内容。这是因为映射的目标词本身被删除，因为它是一个停止词。类似地，如果映射为“bar，foo，baz”，并且`expand`设置为false，则不会添加任何映射，因为`expand=false`时，目标映射是第一个单词。但是，如果`expand=true`，则添加的映射将等效于`foo，baz=>foo，baz`，即除了停止字之外的所有映射。

##不推荐使用`tokenizer`和`ignore_case`

`tokenizer`参数控制将用于标记同义词的标记器，该参数用于向后兼容在6.0之前创建的索引。`ignore_case`参数仅适用于`tokenizer`参数。

支持两种同义词格式：Solr、WordNet。

##Solr同义词

以下是该文件的示例格式：

```json
# Blank lines and lines starting with pound are comments.

# Explicit mappings match any token sequence on the LHS of "=>"
# and replace with all alternatives on the RHS.  These types of mappings
# ignore the expand parameter in the schema.
# Examples:
i-pod, i pod => ipod,
sea biscuit, sea biscit => seabiscuit

# Equivalent synonyms may be separated with commas and give
# no explicit mapping.  In this case the mapping behavior will
# be taken from the expand parameter in the schema.  This allows
# the same synonym file to be used in different synonym handling strategies.
# Examples:
ipod, i-pod, i pod
foozball , foosball
universe , cosmos
lol, laughing out loud

# If expand==true, "ipod, i-pod, i pod" is equivalent
# to the explicit mapping:
ipod, i-pod, i pod => ipod, i-pod, i pod
# If expand==false, "ipod, i-pod, i pod" is equivalent
# to the explicit mapping:
ipod, i-pod, i pod => ipod

# Multiple synonym mapping entries are merged.
foo => foo bar
foo => baz
# is equivalent to
foo => foo bar, baz
```

您还可以直接在配置文件中定义过滤器的同义词（注意使用`synonyms`而不是`synonyms_path`）：

```json
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "filter": {
          "synonym": {
            "type": "synonym",
            "synonyms": [
              "i-pod, i pod => ipod",
              "universe, cosmos"
            ]
          }
        }
      }
    }
  }
}
```

但是，建议使用`synonyms_path`在文件中定义大量的同义词集，因为内联指定它们会不必要地增加集群大小。

##WordNet同义词

基于[WordNet]()格式的同义词可以使用以下`format`声明：

```json
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "filter": {
          "synonym": {
            "type": "synonym",
            "format": "wordnet",
            "synonyms": [
              "s(100000001,1,'abstain',v,1,0).",
              "s(100000001,2,'refrain',v,1,0).",
              "s(100000001,3,'desist',v,1,0)."
            ]
          }
        }
      }
    }
  }
}
```

一样支持使用`synonyms_path`在文件中定义WordNet同义词。

##解析同义词文件

Elasticsearch将使用标记器链中同义词过滤器前面的标记过滤器来解析同义词文件中的条目。因此，例如，如果同义词过滤器放在词干分析器之后，那么词干分析器也将应用于同义词条目。由于同义词映射中的条目不能具有堆叠位置，因此某些标记过滤器可能会导致此处出现问题。生成一个标记的多个版本的标记过滤器可以选择在解析同义词时发出哪个版本的标记，例如，`asciifolding`将只生成该标记的折叠版本。其他，例如`multiplexer`、`word_delimiter_graph`或`ngram`将抛出错误。

如果需要建立包含多个标记过滤器和同义词过滤器的分词器，请考虑使用[`multiplexer`]()过滤器，其中一个分支中使用多个令牌过滤器，另一个分支中使用同义词过滤器。
