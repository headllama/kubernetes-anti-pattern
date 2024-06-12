# Navegando com eficiência no Kubernetes: Evitando Anti-Padrões para Implantações Robustas

O [Kubernetes](https://kubernetes.io/pt-br/) se tornou uma ferramenta fundamental para a implantação e gerenciamento de aplicativos em contêineres em ambientes de nuvem. Sua adoção generalizada reflete sua capacidade de oferecer escalabilidade, agilidade e alta disponibilidade. No entanto, como qualquer tecnologia complexa, o Kubernetes apresenta certos padrões de uso que podem levar a resultados indesejados. Esses padrões, conhecidos como anti-padrões, podem comprometer a performance, a segurança e a confiabilidade das implantações.

Este artigo tem como objetivo apresentar os principais anti-padrões do Kubernetes e fornecer recomendações para evitá-los. Ao adotar práticas recomendadas, as equipes de desenvolvimento e operações podem garantir que suas implantações em Kubernetes sejam robustas, eficientes e operem de forma otimizada.


## **Usar tag "Latest"**

Por que a tag "latest" é problemática?

À primeira vista, a tag "latest" pode parecer conveniente, pois sempre puxa a versão mais recente da imagem do container disponível no [registry](https://hub.docker.com/). No entanto, essa facilidade aparente esconde armadilhas que podem prejudicar a estabilidade e a confiabilidade de suas implantações:

* Dificuldade de controle de versão: Usar a tag "latest" torna difícil rastrear qual versão exata da imagem do container está sendo executada em seu ambiente. Isso dificulta a identificação de problemas específicos e a realização de rollbacks para versões anteriores estáveis.

* Implantação imprevisível: Novas versões de imagens podem conter alterações significativas ou correções de bugs. Ao depender da tag "latest", você corre o risco de que um rollout automático introduza comportamentos inesperados em seu aplicativo.

* Instabilidade potencial: Com a tag "latest", seus pods podem tentar atualizar a imagem do container a qualquer momento, potencialmente causando reinicializações desnecessárias e instabilidade em seu aplicativo.

**Melhores práticas para versionamento de imagens:**

Para garantir a estabilidade e a previsibilidade de suas implantações, é altamente recomendável adotar as seguintes práticas:

Use tags semânticas: Empregue um sistema de [versionamento semântico](https://semver.org/) (ex: v1.2.3) para identificar claramente as diferentes versões de suas imagens de container. Isso permite um melhor controle e facilita o rollback para versões anteriores, se necessário.

Utilize hashes Git: Para máxima precisão, considere usar [hashes Git](https://git-scm.com/docs/git-hash-object) como identificadores de versão (ex: sha256:abc123). Isso garante que uma versão específica da imagem seja sempre utilizada, independentemente de atualizações posteriores no registro.

Implemente pipelines de CI/CD: Integre pipelines de integração contínua e entrega contínua (CI/CD) em seu processo de desenvolvimento. Isso permite automatizar o versionamento e o deploy de imagens, garantindo a consistência e a previsibilidade das implantações.


## **Ignorar Health Checks**

Uma implantação bem-sucedida no Kubernetes depende não apenas de container saudáveis, mas também da capacidade de identificá-los. É aqui que entram as verificações de integridade, também conhecidas como "health checks". Omitir essas verificações, um anti-padrão comum, pode levar a problemas sérios para a disponibilidade e resiliência de seus aplicativos.

Omitir as verificações de integridade pode ter consequências graves como:

* Indisponibilidade oculta: Sem verificações, o Kubernetes pode continuar direcionando tráfego para container não responsivos, resultando em falhas na aplicação e uma experiência ruim para o usuário.

* Escalonamento ineficaz: O Kubernetes pode tentar escalar horizontalmente para lidar com suposta demanda, mesmo que a causa raiz seja container não responsivos. Isso desperdiça recursos e aumenta custos.

**Tipos de Verificações de Integridade:**

O Kubernetes oferece dois tipos principais de [verificações de integridade](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#types-of-probe):

Liveness Probes: Verificam se um container está em execução e pronto para processar solicitações. Um contêiner com falhas repetidas na Liveness Probe será reiniciado pelo Kubernetes.

Readiness Probes: Verificam se um container está totalmente inicializado e pronto para receber tráfego. O Kubernetes remove pods do balanceamento de carga até que a Readiness Probe indique que o container está saudável.

## **Utilizar o namespace default para todos os pods**

Por definição, o Kubernetes cria automaticamente um [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) chamado "default" durante a inicialização do cluster. Este namespace funciona como um espaço lógico genérico para abrigar qualquer recurso que não tenha um namespace explícito definido em sua manifestação do YAML. Pense nele como um depósito de objetos perdidos no Kubernetes, onde recursos soltos aguardam a atribuição de um namespace apropriado.

**Conveniência inicial; risco a longo prazo**

Embora o namespace "default" possa parecer conveniente no início, sua utilização excessiva pode trazer riscos à organização e segurança do seu cluster:

* Falta de Clareza: Com todos os recursos agrupados no "default", fica difícil identificar a qual projeto ou equipe cada recurso pertence, dificultando a gestão e a atribuição de responsabilidades.

* Conflito de Nomes: A ausência de namespaces aumenta a probabilidade de conflitos de nomes entre recursos criados por equipes diferentes. Imagine duas equipes criando Deployments com o mesmo nome, ambas no namespace "default". Isso pode levar a comportamentos inesperados e problemas de implantação.

* Limitações de Segurança: O namespace "default" geralmente possui permissões amplas para permitir acesso básico a recursos. Isso pode ser um problema de segurança se você precisar restringir o acesso a recursos específicos para determinadas equipes ou projetos.

**Melhores práticas para evitar o "default":**

Para evitar as desvantagens do namespace "default", adote as seguintes práticas recomendadas:

Sempre defina namespaces: Sempre especifique um namespace apropriado em todas as manifestações do YAML para seus recursos. Isso melhora a organização, a clareza e a rastreabilidade de seus recursos.

Crie namespaces por projeto/equipe: Implemente uma convenção de nomenclatura para criar namespaces dedicados para cada projeto ou equipe. Isso facilita a separação de preocupações e a atribuição de propriedade.

Limite o uso do "default": Reserve o namespace "default" apenas para recursos temporários ou de teste, e não como um depósito para todos os seus recursos de produção.

## **Loadbalancer em todos os services**

Se você expor o services kubernetes como um tipo: [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer), a cloud - no caso - provisionará um ip externo e esses recursos poderão ficar caros (endereço IPv4 estático externo, computação, preço por segundo…) à medida que você cria muitos dos eles.

Embora pareça tentador expor todos os serviços HTTP usando um LoadBalancer, essa abordagem pode trazer consequências negativas:

* Complexidade Aumentada: Manter e gerenciar um grande número de LoadBalancers espalhados por vários services HTTP pode rapidamente se tornar complexo e propenso a erros.

* Falhas de segurança: Expor serviços internos através de um LoadBalancer pode aumentar a superfície de ataque do seu cluster.

Nem todo serviço HTTP requer a acessibilidade direta pela internet. Considere estas alternativas para um balanceamento de carga inteligente:

[NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport): Essa opção expõe o serviço através de uma porta específica em todos os nós do cluster. É útil para serviços internos acessíveis por outros serviços dentro do cluster, mas não precisa ser acessível externamente.

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/): O Ingress funciona como um gateway de entrada único para serviços HTTP no Kubernetes. Você pode definir regras de roteamento para direcionar o tráfego para diferentes serviços com base em nomes de host, caminhos de URL ou outros critérios. Isso é ideal para expor vários serviços HTTP através de um único endpoint externo.

Services internos: Para services puramente internos que não precisam de acesso externo, use o tipo de service "ClusterIP". Isso cria um endpoint virtual acessível apenas por outros pods dentro do cluster.

## Conclusão
Este guia serve como base para sua jornada neste universo em constante expansão. Mantenha-se atualizado com as melhores práticas da comunidade, participe de fóruns e grupos de discussão, e explore ferramentas e recursos disponíveis para aprimorar suas habilidades.

Com aprendizado contínuo e busca por soluções inovadoras, você estará apto a navegar com segurança e eficiência no mar do Kubernetes, construindo e gerenciando aplicações distribuídas que atendem às suas necessidades de forma robusta, escalável e segura.

Este guia é apenas o início. Ele o convida a uma jornada de aprendizado e aperfeiçoamento contínuos, permitindo que você domine as ferramentas e estratégias necessárias para construir e gerenciar infraestruturas Kubernetes de alto desempenho.

Fontes: 
https://kubernetes.io
https://cloudification.io/cloud-blog/kubernetes-anti-patterns-what-not-to-do/
https://medium.com/@seifeddinerajhi/most-common-mistakes-to-avoid-when-using-kubernetes-anti-patterns-%EF%B8%8F-f4d37586528d
