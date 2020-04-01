## Implementando o ViewPager do Pad Numérico da Calculadora
Este é um dos elementos mais interessantes de todo o projeto quando o assunto é UI. Trata-se do pad/teclado onde se encontram todos os botões numéricos e botões de funções matemáticas avançadas. O mais interessante é a interação responsável por exibir o painel de operações avançadas que fica à direita da tela (quando o dispositivo se encontra na orientação retrato), onde você desliza o painél para a esquerda e são exibidos todos os botões pertinentes às operações matemáticas mais avançadas como seno, coseno, mudança de graus para radianos e etc. O painél numérico padrão fica esmaecido e sobreposto por esse painél avançado e se desejar voltar a vê-lo, basta deslizar novamente o painel avançado para a posição original. Todo esse comportamento é alcançado graças a um componente chamado de `ViewPager`. 

## ViewPager
O ViewPager é um `LayoutManager` que por si só não é uma forma de navegação propriamente dita, mas ele geralmente é utilizado com o TabLayout. Ele pode ser usado separadamente e em qualquer lugar da tela. O ViewPager é usado com mais frequência em conjunto com o `Fragment`, que é uma maneira conveniente de fornecer e gerenciar o ciclo de vida de cada página. Existem adaptadores padrão implementados para o uso de fragmentos com o ViewPager, que abrangem os casos de uso mais comuns. Estes são `FragmentPagerAdapter` e `FragmentStatePagerAdapter`; cada uma dessas classes possui código simples, mostrando como criar uma interface de usuário completa com elas.

## CalculatorPadViewPager
Essa classe estende a classe ViewPager padrão e especializa algumas de suas funcionalidades e comportamentos além de fornecer algoritmos específicos para evitar que alguns botões do painel numérico padrão sejam pressionados de forma errônea quando o painel avançado de operações estiver aberto. 

```java
public class CalculatorPadViewPager extends ViewPager {

    private final PagerAdapter mStaticPagerAdapter = new PagerAdapter() {
        @Override
        public int getCount() {
            return getChildCount();
        }

        @Override
        public View instantiateItem(ViewGroup container, int position) {
            return getChildAt(position);
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            removeViewAt(position);
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public float getPageWidth(int position) {
            return position == 1 ? 7.0f / 9.0f : 1.0f;
        }
    };

    private final OnPageChangeListener mOnPageChangeListener = new SimpleOnPageChangeListener() {
        @Override
        public void onPageSelected(int position) {
            for (int i = getChildCount() - 1; i >= 0; --i) {
                final View child = getChildAt(i);
                // Prevent clicks and accessibility focus from going through to descendants of
                // other pages which are covered by the current page.
                child.setClickable(i == position);
                child.setImportantForAccessibility(i == position
                        ? IMPORTANT_FOR_ACCESSIBILITY_AUTO
                        : IMPORTANT_FOR_ACCESSIBILITY_NO_HIDE_DESCENDANTS);
            }
        }
    };

    private final PageTransformer mPageTransformer = new PageTransformer() {
        @Override
        public void transformPage(View view, float position) {
            if (position < 0.0f) {
                // Pin the left page to the left side.
                view.setTranslationX(getWidth() * -position);
                view.setAlpha(Math.max(1.0f + position, 0.0f));
            } else {
                // Use the default slide transition when moving to the next page.
                view.setTranslationX(0.0f);
                view.setAlpha(1.0f);
            }
        }
    };

    public CalculatorPadViewPager(Context context) {
        this(context, null /* attrs */);
    }

    public CalculatorPadViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);

        setAdapter(mStaticPagerAdapter);
        setBackgroundColor(Color.BLACK);
        setPageMargin(getResources().getDimensionPixelSize(R.dimen.pad_page_margin));
        setPageTransformer(false, mPageTransformer);
        addOnPageChangeListener(mOnPageChangeListener);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        // Invalidate the adapter's data set since children may have been added during inflation.
        getAdapter().notifyDataSetChanged();

        // Let page change listener know about our initial position.
        mOnPageChangeListener.onPageSelected(getCurrentItem());
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean shouldIntercept = super.onInterceptTouchEvent(ev);

        // Only allow the current item to receive touch events.
        if (!shouldIntercept && ev.getActionMasked() == MotionEvent.ACTION_DOWN) {
            final int x = (int) ev.getX() + getScrollX();
            final int y = (int) ev.getY() + getScrollY();

            final int childCount = getChildCount();
            for (int i = childCount - 1; i >= 0; --i) {
                final int childIndex = getChildDrawingOrder(childCount, i);
                final View child = getChildAt(childIndex);
                if (child.getVisibility() == View.VISIBLE
                        && x >= child.getLeft() && x < child.getRight()
                        && y >= child.getTop() && y < child.getBottom()) {
                    shouldIntercept = (childIndex != getCurrentItem());
                    break;
                }
            }
        }

        return shouldIntercept;
    }
}
```
### Habilitando e desabilitando os painéis 
**Nota**: Em versões futuras dessa calculadora esse painel avançado fica disposto acima do painel numérico padrão e não há essa interação de deslizar para a esquerda.   

Aqui está presente o algoritmo que impede que o painél numérico padrão, obtenha foco de serviços de acessibilidade enquanto ele estiver sobreposto pelo painél avançado e também impede cliques acidentais a esse painél. Quando uma página é selecionada o método `onPageSelected(int position)` da classe `SimpleOnPageChangeListener` é invocado e o algoritmo é executado e inicia-se um ciclo `for`. Na parte iterativa do `for` a variável `i` recebe o valor retornado por `getChildCount()` que nesse primeiro momento deve ser o valor inteiro 2 pois há duas filhas disponíveis o painél numérico padrão com índice 0 e o painél avançado com índice 1, mas ele subtraí 1 para que apenas as páginas de índice 1 e 0 sejam selecionadas pelo algoritmo. **Lembre-se que os índices na maioria das linguagens de programação começam em 0.** Na parte condicional do `for` enquanto `i` for maior ou igual a 0 o valor de `i` é decrementado e o `for` roda regressivamente. 

O método `getChildAt(int index)` da classe `ViewGroup` devolve a `view` filha na posição especificada pelo argumento `index` e atribuímos ela a referência `final` `child`. Com a view em mãos basta configurar se ela é clicável com o método `setClickable(boolean flag)` e se ela pode obter foco dos serviços de acessibilidade com o método `setImportantFroAccessibility(boolean flag)`. O método `setImportantForAcessibility(int mode)` define se uma view é importante para acessibilidade, ou seja, se ela dispara eventos de acessibilidade e se é relatada aos serviços de acessibilidade que consultam a tela como o TalkBack do Android. Já o `setClickable(boolean flags)` habilita ou desabilita a interação de clique da view selecionada. Falando dos argumentos do primeiro método `setClickable()`, se `i` não for igual a `position` a view selecionada não é mais clicável e nem obtém foco dos serviços de acessibilidade pois `setImportantForAccessibility(int mode)`configura a view com uma das duas constantes disponíveis no operador ternário `IMPORTANT_FOR_ACCESSIBILITY_AUTO` ou `IMPORTANT_FOR_ACCESSIBILITY_NO_HIDE_DESCENDANTS`, caso contrário, ela se torna clicável novamente e pode obter foco do serviço de acessibilidade. Ou seja, se a filha selecionada pelo método `onPageSelected(int index)` for o painél numérico ela se torna clicável e disponível para os serviços de acessibilidade mas se o painél avançado estiver sobreposto ao painél padrão é ele quem se torna disponível aos serviços de acessibilidade e também é clicável. Os dois nunca estarão acessíveis ao mesmo tempo.        

```java
private final OnPageChangeListener mOnPageChangeListener = new SimpleOnPageChangeListener() {
        @Override
        public void onPageSelected(int position) {
            for (int i = getChildCount() - 1; i >= 0; --i) {
                final View child = getChildAt(i);
                // Prevent clicks and accessibility focus from going through to descendants of
                // other pages which are covered by the current page.
                child.setClickable(i == position);
                child.setImportantForAccessibility(i == position
                        ? IMPORTANT_FOR_ACCESSIBILITY_AUTO
                        : IMPORTANT_FOR_ACCESSIBILITY_NO_HIDE_DESCENDANTS);
            }
        }
    };
```
### Constantes de acessibilidade da classe `View`
Como foi visto no método `setImportantForAccessibility(int mode)` é possível configurar uma `view` para ser lida por mecanismos de acessibilidade utilizando uma das constantes disponíveis para essa finalidade: 

* `public static final int IMPORTANT_FOR_ACCESSIBILITY_NO_HIDE_DESCENDANTS` - A view não é importante para acessibilidade nem nenhuma view descendentes 
* `public static final int IMPORTANT_FOR_ACCESSIBILITY_AUTO` - O sistema automaticamente determina quando uma view é importante para acessibilidade 

### PageTransformer
Para realizar a ação de sobrepor o painél numérico padrão é utilizado um `PageTransformer`. Um `PageTransformer` pode personalizar a animação quando uma página é deslizada. Essa interface expõe apenas um método `transformPage()`. Em cada ponto da transição da tela, esse método será chamado uma vez para cada página visível (geralmente há apenas uma página visível) e para páginas adjacentes fora da tela. Por exemplo, se a página três estiver visível e o usuário arrastar para a página quatro, transformPage() será chamado para as páginas dois, três e quatro em cada etapa do gesto.

Na sua implementação do transformPage(), você poderá criar animações de deslize personalizadas determinando quais páginas precisam ser transformadas com base na posição da página na tela, que é conseguida a partir do parâmetro position do método transformPage().

O parâmetro position indica onde uma determinada página está localizada em relação ao centro da tela. Esse parâmetro é uma propriedade dinâmica que muda à medida que o usuário rola pelas páginas. Quando uma página preenche a tela, o valor de posição dela é 0. Quando uma página é arrastada levemente para o lado direito da tela, o valor de posição dela é 1. Se o usuário rolar metade do caminho entre as páginas um e dois, a página um terá uma posição de -0,5 e a página dois terá uma posição de 0,5. Com base na posição das páginas na tela, você poderá criar animações de deslize personalizadas definindo propriedades da página com métodos como setAlpha(), setTranslationX() ou setScaleY().

Quando você tiver uma implementação de um `PageTransformer`, chame `setPageTransformer()` com essa implementação para aplicar as animações personalizadas. É desse modo que obtemos uma animação de deslizar exclusiva e bem útil em nosso painél avançado.  
```java
  private final PageTransformer mPageTransformer = new PageTransformer() {
        @Override
        public void transformPage(View view, float position) {
            if (position < 0.0f) {
                // Pin the left page to the left side.
                view.setTranslationX(getWidth() * -position);
                view.setAlpha(Math.max(1.0f + position, 0.0f));
            } else {
                // Use the default slide transition when moving to the next page.
                view.setTranslationX(0.0f);
                view.setAlpha(1.0f);
            }
        }
    };
```  
