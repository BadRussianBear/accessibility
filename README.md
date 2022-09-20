# модуль accessibility
# файлы AccessibilityExtension, ConstraintLayoutAccessibilityView

Если мы хотим указать, что компонент view не важен для accessibility (не требуется возможность установки на нем фокуса и, в следствие чего, его 
contentDescription не будет озвучен), то в разметке такие компоненты помечаем в виде
android:importantForAccessibility="no"
<com.google.android.material.textview.MaterialTextView
    android:id="@+id/mtv3"
    android:importantForAccessibility="no" ... />

Если мы хотим указать, что компонент родитель view и все его вложенные view не важны для accessibility, то в разметке такие компоненты помечаем в виде
android:importantForAccessibility="noHideDescendants"

<androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/parentView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:importantForAccessibility="noHideDescendants">
        
        <com.google.android.material.textview.MaterialTextView
            android:id="@+id/mtv1" ... />

        <com.google.android.material.textview.MaterialTextView
            android:id="@+id/mtv2" ... />

        <com.google.android.material.textview.MaterialTextView
            android:id="@+id/mtv3" ... />
</androidx.constraintlayout.widget.ConstraintLayout>

/----------------------/

/**
 * функция расширение, возвращающая AccessibilityNodeInfoCompat для установки параметров accessibility
 * принимает в себя лямбду
 */
 
fun View.setupAccessibilityInfo(infoAction: (info: AccessibilityNodeInfoCompat) -> Unit) {
    ViewCompat.setAccessibilityDelegate(this, object : AccessibilityDelegateCompat() {
        override fun onInitializeAccessibilityNodeInfo(
            host: View,
            info: AccessibilityNodeInfoCompat
        ) {
            super.onInitializeAccessibilityNodeInfo(host, info)
            infoAction(info)
        }
    })
}

Способ использования:
binding.yourViewId.setupAccessibilityInfo { info ->
    
}

/----------------------/

/**
 * функция расширение, возвращающая AccessibilityEvent для установки параметров accessibility
 * принимает в себя лямбду
 */
fun View.setupAccessibilityEvent(eventAction: (event: AccessibilityEvent) -> Unit) {
    ViewCompat.setAccessibilityDelegate(this, object : AccessibilityDelegateCompat() {
        override fun onInitializeAccessibilityEvent(host: View?, event: AccessibilityEvent) {
            super.onInitializeAccessibilityEvent(host, event)
            eventAction(event)
        }
    })
}

Способ использования:
binding.yourViewId.setupAccessibilityEvent { event ->
    
}

/----------------------/

Указать текст для озвучки в случае, если contentDescription для вью должен отличаться от contentDescription для озвучки
info.contentDescription = "your text for talkback" 

В случае, если мы хотим озвучить компонент view "кнопка" отлично от существующего шаблона "Имя кнопки. Кнопка" в виде "Кнопка - имя кнопки", то мы можем
"подменить" класс компонента view и задать необходимый текст:
info.className = Image::class.java.name
info.contentDescription = "Кнопка - имя кнопки" 

/----------------------/

В случае, если у нас сложилась такая ситуация, что произошло некое соыбтие, но отсутствует компонент view, который мы можем выставить в фокус,
то можем вызвать данную функцию и передать в нее текст для озвучки
fun Context.announceForAccessibility(announcement: String) {
    this
        .getSystemService(Context.ACCESSIBILITY_SERVICE)
        .let { it as AccessibilityManager }
        .let { manager ->
            AccessibilityEvent
                .obtain()
                .apply {
                    eventType = AccessibilityEvent.TYPE_ANNOUNCEMENT
                    className = this@announceForAccessibility.javaClass.name
                    packageName = this@announceForAccessibility.packageName
                    text.add(announcement)
                }
                .let {
                    manager.sendAccessibilityEvent(it)
                }
        }
}

/----------------------/

Если у нас есть экран, построеный на основе ConstraintLayout и мы не хотим создавать дополнительных вложенностей, но хотим выделить отдельные группы view
с определенной зоной покрытия (перехватывать события тапа по участку экрана и озвучивать определенную информацию), то можем использовать 
ConstraintLayoutAccessibilityView.

Использование:
<ru.minsvyaz.accessibility.ConstraintLayoutAccessibilityView
    android:id="@+id/fpClavBill"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="@+id/fpTvBillDesc"
    app:layout_constraintTop_toTopOf="@+id/fpTvPaymentTitle"
    app:constraint_referenced_ids="fpTvPaymentTitle,fpTvBillDateDesc,fpTvBillDesc"
    tools:visibility="invisible" />
    
В constraint_referenced_ids передаются id, чьи ocntentDescription-ы будут озвучены при тапе по экрану в пределах ConstraintLayoutAccessibilityView.
ЗЫ: зачитываться будут только те компоненты, которые видимы, т.е. view с параметром android:visibility="gone" будет проигнорировано.
В случае необходимости, можно так же задать всем view android:importantForAccessibility="no" и переназначить contentDescription для ConstraintLayoutAccessibilityView

binding.fpClavBill.setupAccessibilityInfo { info ->
    info.contentDescription = "Your text" 
}

/----------------------/

Если в компоненте view установлен текст, включающий в себя жесткий перевод строки \n, то необходимо переназначить его contentDescription пропустив строку
через функцию clearNewLines(), описанную в файле AccessibilityExtension модуля accessibility

val message = text.clearNewLines()
ffClHint.setupAccessibilityInfo { info ->
    info.contentDescription = "$message"
}
