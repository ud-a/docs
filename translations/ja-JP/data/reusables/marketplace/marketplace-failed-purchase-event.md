---
ms.openlocfilehash: 744983c086ce7f67bb25cd9508e080ceb12ea517
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2022
ms.locfileid: "145123181"
---
顧客がプランをアップグレードし、支払いに失敗した場合、GitHubはその顧客の{% data variables.product.prodname_marketplace %}サブスクリプションを以前の状態に戻します。 また、GitHubはその顧客に対して失敗を知らせ、購入を再度試みることができるよう、メールを送信します。 以前のプランに戻すように要求する `changed` アクションを含む Webhook を受け取ります。
