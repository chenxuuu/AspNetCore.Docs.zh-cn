---
ms.openlocfilehash: 1f8d3913c83aaf5fe6ec2cec482a30f0f066c16b
ms.sourcegitcommit: 34bf9fc6ea814c039401fca174642f0acb14be3c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/14/2019
ms.locfileid: "57841688"
---

> [!NOTE]
> <span data-ttu-id="8eefe-101">在本教程中，使用 Entity Framework Core 迁移功能（若可行）。</span><span class="sxs-lookup"><span data-stu-id="8eefe-101">For this tutorial you use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="8eefe-102">迁移会更新数据库架构，使其与数据模型中的更改相匹配。</span><span class="sxs-lookup"><span data-stu-id="8eefe-102">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="8eefe-103">但是，迁移仅能执行 EF Core 提供程序所支持的更改类型，且 SQLite 提供程序的功能将受限。</span><span class="sxs-lookup"><span data-stu-id="8eefe-103">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="8eefe-104">例如，支持添加列，但不支持删除或更改列。</span><span class="sxs-lookup"><span data-stu-id="8eefe-104">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="8eefe-105">如果已创建迁移以删除或更改列，则 `ef migrations add` 命令将成功，但 `ef database update` 命令会失败。</span><span class="sxs-lookup"><span data-stu-id="8eefe-105">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="8eefe-106">由于上述限制，本教程不对 SQLite 架构更改使用迁移。</span><span class="sxs-lookup"><span data-stu-id="8eefe-106">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="8eefe-107">转而在架构更改时，放弃并重新创建数据库。</span><span class="sxs-lookup"><span data-stu-id="8eefe-107">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="8eefe-108">要绕开 SQLite 限制，可手动写入迁移代码，在表内容更改时重新生成表。</span><span class="sxs-lookup"><span data-stu-id="8eefe-108">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="8eefe-109">表重新生成涉及：</span><span class="sxs-lookup"><span data-stu-id="8eefe-109">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="8eefe-110">重命名现有表。</span><span class="sxs-lookup"><span data-stu-id="8eefe-110">Renaming the existing table.</span></span>
>* <span data-ttu-id="8eefe-111">创建新表。</span><span class="sxs-lookup"><span data-stu-id="8eefe-111">Creating a new table.</span></span>
>* <span data-ttu-id="8eefe-112">将旧表中的数据复制到新表中。</span><span class="sxs-lookup"><span data-stu-id="8eefe-112">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="8eefe-113">放弃旧表。</span><span class="sxs-lookup"><span data-stu-id="8eefe-113">Dropping the old table.</span></span>
>
><span data-ttu-id="8eefe-114">有关更多信息，请参见以下资源：</span><span class="sxs-lookup"><span data-stu-id="8eefe-114">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="8eefe-115">SQLite EF Core 数据库提供程序限制</span><span class="sxs-lookup"><span data-stu-id="8eefe-115">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="8eefe-116">自定义迁移代码</span><span class="sxs-lookup"><span data-stu-id="8eefe-116">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="8eefe-117">数据种子设定</span><span class="sxs-lookup"><span data-stu-id="8eefe-117">Data seeding</span></span>](/ef/core/modeling/data-seeding)