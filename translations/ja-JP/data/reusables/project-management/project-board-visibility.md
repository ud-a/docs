---
ms.openlocfilehash: 6f5f7b9a1ef172b471215d5ea66d834fb00e19d7
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2022
ms.locfileid: "147423693"
---
既定では、ユーザー所有および Organization 全体の {% data variables.projects.projects_v1_boards %} はプライベートであり、{% data variables.projects.projects_v1_board %} に対する読み取り、書き込み、または管理者権限を持つユーザーにのみ表示されます。 {% ifversion ghae %}内部{% else %}公開{% endif %} {% data variables.projects.projects_v1_board %} は、{% ifversion ghae %} Enterprise に {% else %}すべての{% endif %} {% data variables.projects.projects_v1_board %} の URL を使ってアクセスした人が表示できます。 リポジトリ レベルの {% data variables.projects.projects_v1_boards %} は、リポジトリの可視性を共有します。 つまり、プライベート リポジトリにはプライベート ボードがあり、この可視性を変更することはできません。
