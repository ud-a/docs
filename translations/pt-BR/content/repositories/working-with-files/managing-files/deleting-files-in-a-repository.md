---
title: Excluir arquivos em um repositório
intro: 'Você pode excluir um arquivo individual{% ifversion fpt or ghes or ghec %} ou um diretório inteiro{% endif %} em seu repositório no {% data variables.product.product_name %}.'
redirect_from:
  - /articles/deleting-files
  - /github/managing-files-in-a-repository/deleting-files
  - /github/managing-files-in-a-repository/deleting-a-file-or-directory
  - /github/managing-files-in-a-repository/deleting-files-in-a-repository
  - /github/managing-files-in-a-repository/managing-files-on-github/deleting-files-in-a-repository
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
permissions: 'People with write permissions can delete files{% ifversion fpt or ghes or ghec %} or directories{% endif %} in a repository.'
topics:
  - Repositories
shortTitle: Delete files
ms.openlocfilehash: b3d939a7be6be37e875104f7a3c4df53f7a3b7ed
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/05/2022
ms.locfileid: '145126978'
---
## Sobre a exclusão de arquivo{% ifversion fpt or ghes or ghec %} e de diretório{% endif %}

Você pode excluir um arquivo individual no seu repositório{% ifversion fpt or ghes or ghec %} ou em um diretório inteiro, incluindo todos os arquivos do diretório{% endif %}.

Se você tentar excluir um arquivo{% ifversion fpt or ghes or ghec %} ou um diretório{% endif %} em um repositório no qual você não tenha permissões de gravação, faremos uma bifurcação do projeto em sua conta pessoal e ajudaremos você a enviar uma solicitação de pull para o repositório original depois de você fazer commit da alteração. Para obter mais informações, confira "[Sobre as solicitações de pull](/github/collaborating-with-issues-and-pull-requests/about-pull-requests)".

Se o arquivo{% ifversion fpt or ghes or ghec %} ou o diretório{% endif %} excluído contiver dados confidenciais, os dados ainda estarão disponíveis no histórico do Git do repositório. Para remover completamente o arquivo de {% data variables.product.product_name %}, você deve remover o arquivo do histórico do seu repositório. Para obter mais informações, confira "[Como remover dados confidenciais de um repositório](/github/authenticating-to-github/removing-sensitive-data-from-a-repository)".

## Excluir um arquivo

1. Navegue até o arquivo no repositório que deseja excluir.
2. No início do arquivo, clique em {% octicon "trash" aria-label="The trash icon" %}.
{% data reusables.files.write_commit_message %} {% data reusables.files.choose-commit-email %} {% data reusables.files.choose_commit_branch %} {% data reusables.files.propose_file_change %}

{% ifversion fpt or ghes or ghec %}
## Excluir um diretório

1. Acesse o diretório no seu repositório que deseja excluir.
1. No canto superior direito, clique em {% octicon "kebab-horizontal" aria-label="The horizontal kebab icon" %} e clique em **Excluir diretório**.
  ![Botão usado para excluir um diretório](/assets/images/help/repository/delete-directory-button.png)
1. Revise os arquivos que você excluirá.
{% data reusables.files.write_commit_message %} {% data reusables.files.choose-commit-email %} {% data reusables.files.choose_commit_branch %} {% data reusables.files.propose_file_change %} {% endif %}
