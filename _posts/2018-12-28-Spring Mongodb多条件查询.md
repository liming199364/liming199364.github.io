---
layout: post 
title: 'Spring Mongodb多条件查询'
date: 2018-12-21
categories: Spring
tags: Spring
---

### 比较操作符

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$eq`](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq) | Matches values that are equal to a specified value.          |
| [`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt) | Matches values that are greater than a specified value.      |
| [`$gte`](https://docs.mongodb.com/manual/reference/operator/query/gte/#op._S_gte) | Matches values that are greater than or equal to a specified value. |
| [`$in`](https://docs.mongodb.com/manual/reference/operator/query/in/#op._S_in) | Matches any of the values specified in an array.             |
| [`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) | Matches values that are less than a specified value.         |
| [`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) | Matches values that are less than or equal to a specified value. |
| [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne) | Matches all values that are not equal to a specified value.  |
| [`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin) | Matches none of the values specified in an array.            |

### 逻辑操作符

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) | Joins query clauses with a logical `AND` returns all documents that match the conditions of both clauses. |
| [`$not`](https://docs.mongodb.com/manual/reference/operator/query/not/#op._S_not) | Inverts the effect of a query expression and returns documents that do *not* match the query expression. |
| [`$nor`](https://docs.mongodb.com/manual/reference/operator/query/nor/#op._S_nor) | Joins query clauses with a logical `NOR` returns all documents that fail to match both clauses. |
| [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or) | Joins query clauses with a logical `OR` returns all documents that match the conditions of either clause. |

### 实际需求

工作中在做用户画像，通过Spark计算得到用户基础标签后，前端会需要组合各种基础标签进行多条件查询，最基本的用法是通过逻辑操作符，操作当前空间下的比较操作符得到结果，mongdb支持多条件查询，对应Spring开发则需要依赖

```
package org.bson.conversions;
```

粘贴下Bson里面的注释

```java
/**
 * An interface for types that are able to render themselves into a {@code BsonDocument}.
 *
 * @since 3.0
 */
public interface Bson {
    /**
     * Render the filter into a BsonDocument.
     *
     * @param documentClass the document class in scope for the collection.  This parameter may be ignored, but it may be used to alter
     *                      the structure of the returned {@code BsonDocument} based on some knowledge of the document class.
     * @param codecRegistry the codec registry.  This parameter may be ignored, but it may be used to look up {@code Codec} instances for
     *                      the document class or any other related class.
     * @param <TDocument> the type of the document class
     * @return the BsonDocument
     */
    <TDocument> BsonDocument toBsonDocument(Class<TDocument> documentClass, CodecRegistry codecRegistry);
}
```

### 具体实现

```java
    /**
     * Finds all documents in the collection.
     *
     * @param filter the query filter
     * @return the find iterable interface
     * @mongodb.driver.manual tutorial/query-documents/ Find
     */
    FindIterable<TDocument> find(Bson filter);
```

Mongodb内部支持通过Bson filter进行多条件查询，返回TDocument对象

所以，需要通过用户自定义的text生成Bson filter，这里提供我内部使用的工具类，后续方便使用

```java
import org.bson.Document;
import org.bson.conversions.Bson;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static com.mongodb.client.model.Filters.*;

@Component
public class ConditionExprToMongoFilterImpl implements ConditionExprToMongoFilter {
    private static Map<String, ConditionExprToMongoFilter> map;
    private static ConditionExprToMongoFilterImpl conditionExprToMongoFilter = new ConditionExprToMongoFilterImpl();

    static {
        map = new HashMap<>();
        map.put("and", (expr) -> {
                List<Bson> bsons = new ArrayList<>();
                for (ConditionExpr subExpr : expr.getSubExpr() ) {
                    bsons.add(conditionExprToMongoFilter.getMongoFilter(subExpr));
                }
                return and(bsons);
        });
        map.put("or", (expr) -> {
                List<Bson> bsons = new ArrayList<>();
                for (ConditionExpr subExpr : expr.getSubExpr() ) {
                    bsons.add(conditionExprToMongoFilter.getMongoFilter(subExpr));
                }
                return or(bsons);
        });
        map.put("not", (expr) -> not(conditionExprToMongoFilter.getMongoFilter(expr.getSubExpr().get(0))));
        map.put("nand", (expr) -> {
            List<Bson> bsons = new ArrayList<>();
            for (ConditionExpr subExpr : expr.getSubExpr() ) {
                bsons.add(conditionExprToMongoFilter.getMongoFilter(subExpr));
            }
            return not(and(bsons));
        });
        map.put("nor", (expr) -> {
            List<Bson> bsons = new ArrayList<>();
            for (ConditionExpr subExpr : expr.getSubExpr() ) {
                bsons.add(conditionExprToMongoFilter.getMongoFilter(subExpr));
            }
            return nor(bsons);
        });

        map.put("string", (expr)-> new Document(expr.getField(), expr.getValue().get(0)));
        map.put("int", (expr)-> new Document(expr.getField(), Integer.valueOf(expr.getValue().get(0))));
        map.put("double", (expr)-> new Document(expr.getField(), Double.valueOf(expr.getValue().get(0))));

        map.put("intGt", (expr)-> gt(expr.getField(), Integer.valueOf(expr.getValue().get(0))));
        map.put("intGte", (expr)-> gte(expr.getField(), Integer.valueOf(expr.getValue().get(0))));
        map.put("intLt", (expr)-> lt(expr.getField(), Integer.valueOf(expr.getValue().get(0))));
        map.put("intLte", (expr)-> lte(expr.getField(), Integer.valueOf(expr.getValue().get(0))));

        map.put("intOpenInterval", (expr)-> and(
                gt(expr.getField(), Integer.valueOf(expr.getValue().get(0))),
                lt(expr.getField(), Integer.valueOf(expr.getValue().get(1)))
        ));
        map.put("intCloseInterval", (expr)-> and(
                gte(expr.getField(), Integer.valueOf(expr.getValue().get(0))),
                lte(expr.getField(), Integer.valueOf(expr.getValue().get(1)))
        ));
        map.put("intRightOpenInterval", (expr)-> and(
                gte(expr.getField(), Integer.valueOf(expr.getValue().get(0))),
                lt(expr.getField(), Integer.valueOf(expr.getValue().get(1)))
        ));
        map.put("intLeftOpenInterval", (expr)-> and(
                gt(expr.getField(), Integer.valueOf(expr.getValue().get(0))),
                lte(expr.getField(), Integer.valueOf(expr.getValue().get(1)))
        ));

        map.put("doubleGt", (expr)-> gt(expr.getField(), Double.valueOf(expr.getValue().get(0))));
        map.put("doubleLt", (expr)-> lt(expr.getField(), Double.valueOf(expr.getValue().get(0))));
        map.put("doubleInterval", (expr)-> and(
                gt(expr.getField(), Double.valueOf(expr.getValue().get(0))),
                lt(expr.getField(), Double.valueOf(expr.getValue().get(1)))
                ));
    }

    @Override
    public Bson getMongoFilter(ConditionExpr expr) throws IOException {
        ConditionExprToMongoFilter obj;
        obj = map.get(expr.getOp());
        if (obj == null) {
            throw new IOException("op " + expr.getOp() + "not define.");
        }
        return obj.getMongoFilter(expr);
    }
}
```

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@Scope("prototype")
public class ConditionExpr {
    @JsonProperty(value = "op")
    private String op;

    @JsonProperty(value = "sub")
    private List<ConditionExpr> subExpr;

    @JsonProperty(value = "field")
    private String field;

    @JsonProperty(value = "value")
    private List<String> value;

    public ConditionExpr() {
    }

    public String getOp() {
        return op;
    }

    public void setOp(String op) {
        this.op = op;
    }

    public List<ConditionExpr> getSubExpr() {
        return subExpr;
    }

    public void setSubExpr(List<ConditionExpr> subExpr) {
        this.subExpr = subExpr;
    }

    public String getField() {
        return field;
    }

    public void setField(String field) {
        this.field = field;
    }

    public List<String> getValue() {
        return value;
    }

    public void setValue(List<String> value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "ConditionExpr{" +
                "op='" + op + '\'' +
                ", subExpr=" + subExpr +
                ", field='" + field + '\'' +
                ", value=" + value +
                '}';
    }
}
```

开发者可以按照下面的json定义自定义filter text
```json
{
    "tagGroupName": "性别",
    "tagName": "过滤1",
    "tagExpr": {
        "op": "and",
        "sub": [
            {
                "op": "string",
                "field": "gender",
                "value": [
                    "M"
                ]
            },
            {
                "op": "or",
                "sub": [
                    {
                        "op": "intCloseInterval",
                        "field": "age",
                        "value": [
                            "26",
                            "40"
                        ]
                    },
                    {
                        "op": "intCloseInterval",
                        "field": "age",
                        "value": [
                            "0",
                            "18"
                        ]
                    }
                ]
            }
        ]
    }
}
```

通过ConditionExprToMongoFilterImpl工具类得到Bson filter，然后使用

```
collection.find(filter)
```

就可以得到相应的document对象了~

