O que é o FOMO : FOMO é uma arquitetura de modelo de detecção de objetos. A sigla significa Fast Objects, Mobile, Optimized, que pode ser traduzido como "Objetos Rápidos, Otimizados para Dispositivos Móveis".

Desenvolvida pela empresa Edge Impulse, a arquitetura FOMO é projetada para ser extremamente leve e eficiente, ideal para rodar em microcontroladores e dispositivos com poucos recursos, como o ESP32 e o Raspberry Pi.

Como o FOMO Funciona?
Diferente de modelos tradicionais (como o YOLO), que detectam objetos com caixas delimitadoras, o FOMO detecta apenas o centro de cada objeto e sua classe. Isso reduz drasticamente a complexidade do modelo e o tempo de processamento, tornando-o perfeito para:

* Identificação de um único objeto: Como um sensor de incêndio que precisa saber se há fogo ou não na imagem.

* Detecção de contagem: Para contar o número de pessoas, animais ou objetos em um ambiente.

