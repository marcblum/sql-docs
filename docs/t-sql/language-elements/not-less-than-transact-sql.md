---
title: "!&lt; (Not Less Than) (Transact-SQL)"
description: "!&lt; (Not Less Than) (Transact-SQL)"
ms.prod: sql
ms.prod_service: "database-engine, sql-database"
ms.technology: t-sql
ms.topic: reference
f1_keywords: 
  - "!<"
  - "Not Less"
  - "!<_TSQL"
  - "Not Less Than"
  - "Less Than"
dev_langs: 
  - "TSQL"
helpviewer_keywords: 
  - "!< (not less than)"
  - "not less than operator (!<)"
ms.assetid: ecbb598e-58a2-4b6c-90b4-3ad5bdfcae39
author: LitKnd
ms.author: kendralittle
ms.reviewer: ""
ms.custom: ""
ms.date: "03/13/2017"
---

# !&lt; (Not Less Than) (Transact-SQL)

[!INCLUDE [SQL Server Azure SQL Database Azure SQL Managed Instance](../../includes/applies-to-version/sql-asdb-asdbmi.md)]

Compares two expressions (a comparison operator). When you compare nonnull expressions, the result is TRUE if the left operand does not have a value lower than the right operand; otherwise, the result is FALSE. If either or both operands are NULL, see the topic [SET ANSI_NULLS &#40;Transact-SQL&#41;](../../t-sql/statements/set-ansi-nulls-transact-sql.md).  
  
![Topic link icon](../../database-engine/configure-windows/media/topic-link.gif "Topic link icon") [Transact-SQL Syntax Conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)  
  
## Syntax  
  
```syntaxsql
expression !< expression  
```  
  
[!INCLUDE[sql-server-tsql-previous-offline-documentation](../../includes/sql-server-tsql-previous-offline-documentation.md)]

## Arguments
 *expression*  
 Is any valid [expression](../../t-sql/language-elements/expressions-transact-sql.md). Both expressions must have implicitly convertible data types. The conversion depends on the rules of [data type precedence](../../t-sql/data-types/data-type-precedence-transact-sql.md).  
  
## Result Types  
 **Boolean**  
  
## See Also  
 [Data Types &#40;Transact-SQL&#41;](../../t-sql/data-types/data-types-transact-sql.md)   
 [Operators &#40;Transact-SQL&#41;](../../t-sql/language-elements/operators-transact-sql.md)  
  
  
