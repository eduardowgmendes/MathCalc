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

Em versões futuras dessa calculadora esse painel avançado fica disposto acima do painel numérico padrão e não há essa interação de deslizar para a esquerda.     
