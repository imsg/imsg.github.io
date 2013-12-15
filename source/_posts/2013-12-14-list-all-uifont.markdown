---
layout: post
title: "列举所有UIFont字体"
date: 2013-12-14 23:08
comments: true
categories: [iOS, dev_tips]
---

UI开发中总是需要了解iOS中提供的所有字体，UIFont接口`+ (UIFont *)fontWithName:(NSString *)fontName size:(CGFloat)fontSize
`也是通过字体名称来得到UIFont对象。

简单写了几行代码就可以列举出所有字体。

<!--more-->


```objc
NSString *family, *font;
for (family in [UIFont familyNames]) {
    NSLog(@"Family: %@", family);
    for (font in [UIFont fontNamesForFamilyName:family]) {
        NSLog(@"\t\tFont: %@", font);
    }
}
```

打印出结果如下：

```objc
Family: Thonburi
	Font: Thonburi-Bold
	Font: Thonburi
	Font: Thonburi-Light
Family: Snell Roundhand
	Font: SnellRoundhand-Black
	Font: SnellRoundhand-Bold
	Font: SnellRoundhand
Family: Academy Engraved LET
	Font: AcademyEngravedLetPlain
Family: Marker Felt
	Font: MarkerFelt-Thin
	Font: MarkerFelt-Wide
Family: Avenir
	Font: Avenir-Heavy
	Font: Avenir-Oblique
	Font: Avenir-Black
	Font: Avenir-Book
	Font: Avenir-BlackOblique
	Font: Avenir-HeavyOblique
	Font: Avenir-Light
	Font: Avenir-MediumOblique
	Font: Avenir-Medium
	Font: Avenir-LightOblique
	Font: Avenir-Roman
	Font: Avenir-BookOblique
Family: Geeza Pro
	Font: GeezaPro-Bold
	Font: GeezaPro
	Font: GeezaPro-Light
Family: Arial Rounded MT Bold
	Font: ArialRoundedMTBold
Family: Trebuchet MS
	Font: Trebuchet-BoldItalic
	Font: TrebuchetMS
	Font: TrebuchetMS-Bold
	Font: TrebuchetMS-Italic
Family: Arial
	Font: ArialMT
	Font: Arial-BoldItalicMT
	Font: Arial-ItalicMT
	Font: Arial-BoldMT
Family: Marion
	Font: Marion-Regular
	Font: Marion-Italic
	Font: Marion-Bold
Family: Menlo
	Font: Menlo-BoldItalic
	Font: Menlo-Regular
	Font: Menlo-Bold
	Font: Menlo-Italic
Family: Malayalam Sangam MN
	Font: MalayalamSangamMN
	Font: MalayalamSangamMN-Bold
Family: Kannada Sangam MN
	Font: KannadaSangamMN
	Font: KannadaSangamMN-Bold
Family: Gurmukhi MN
	Font: GurmukhiMN-Bold
	Font: GurmukhiMN
Family: Bodoni 72 Oldstyle
	Font: BodoniSvtyTwoOSITCTT-BookIt
	Font: BodoniSvtyTwoOSITCTT-Bold
	Font: BodoniSvtyTwoOSITCTT-Book
Family: Bradley Hand
	Font: BradleyHandITCTT-Bold
Family: Cochin
	Font: Cochin-Bold
	Font: Cochin-BoldItalic
	Font: Cochin-Italic
	Font: Cochin
Family: Sinhala Sangam MN
	Font: SinhalaSangamMN
	Font: SinhalaSangamMN-Bold
Family: Hiragino Kaku Gothic ProN
	Font: HiraKakuProN-W6
	Font: HiraKakuProN-W3
Family: Iowan Old Style
	Font: IowanOldStyle-Bold
	Font: IowanOldStyle-BoldItalic
	Font: IowanOldStyle-Italic
	Font: IowanOldStyle-Roman
Family: Damascus
	Font: DamascusBold
	Font: Damascus
	Font: DamascusMedium
	Font: DamascusSemiBold
Family: Al Nile
	Font: AlNile-Bold
	Font: AlNile
Family: Farah
	Font: Farah
Family: Papyrus
	Font: Papyrus-Condensed
	Font: Papyrus
Family: Verdana
	Font: Verdana-BoldItalic
	Font: Verdana-Italic
	Font: Verdana
	Font: Verdana-Bold
Family: Zapf Dingbats
	Font: ZapfDingbatsITC
Family: DIN Condensed
	Font: DINCondensed-Bold
Family: Avenir Next Condensed
	Font: AvenirNextCondensed-Regular
	Font: AvenirNextCondensed-MediumItalic
	Font: AvenirNextCondensed-UltraLightItalic
	Font: AvenirNextCondensed-UltraLight
	Font: AvenirNextCondensed-BoldItalic
	Font: AvenirNextCondensed-Italic
	Font: AvenirNextCondensed-Medium
	Font: AvenirNextCondensed-HeavyItalic
	Font: AvenirNextCondensed-Heavy
	Font: AvenirNextCondensed-DemiBoldItalic
	Font: AvenirNextCondensed-DemiBold
	Font: AvenirNextCondensed-Bold
Family: Courier
	Font: Courier
	Font: Courier-Oblique
	Font: Courier-BoldOblique
	Font: Courier-Bold
Family: Hoefler Text
	Font: HoeflerText-Regular
	Font: HoeflerText-BlackItalic
	Font: HoeflerText-Italic
	Font: HoeflerText-Black
Family: Euphemia UCAS
	Font: EuphemiaUCAS
	Font: EuphemiaUCAS-Bold
	Font: EuphemiaUCAS-Italic
Family: Helvetica
	Font: Helvetica-Oblique
	Font: Helvetica-Light
	Font: Helvetica-Bold
	Font: Helvetica
	Font: Helvetica-BoldOblique
	Font: Helvetica-LightOblique
Family: Hiragino Mincho ProN
	Font: HiraMinProN-W6
	Font: HiraMinProN-W3
Family: Bodoni Ornaments
	Font: BodoniOrnamentsITCTT
Family: Superclarendon
	Font: Superclarendon-Regular
	Font: Superclarendon-BoldItalic
	Font: Superclarendon-Light
	Font: Superclarendon-BlackItalic
	Font: Superclarendon-Italic
	Font: Superclarendon-LightItalic
	Font: Superclarendon-Bold
	Font: Superclarendon-Black
Family: Mishafi
	Font: DiwanMishafi
Family: Optima
	Font: Optima-Regular
	Font: Optima-Italic
	Font: Optima-Bold
	Font: Optima-BoldItalic
	Font: Optima-ExtraBlack
Family: Gujarati Sangam MN
	Font: GujaratiSangamMN-Bold
	Font: GujaratiSangamMN
Family: Devanagari Sangam MN
	Font: DevanagariSangamMN
	Font: DevanagariSangamMN-Bold
Family: Apple Color Emoji
	Font: AppleColorEmoji
Family: Savoye LET
	Font: SavoyeLetPlain
Family: Kailasa
	Font: Kailasa
	Font: Kailasa-Bold
Family: Times New Roman
	Font: TimesNewRomanPS-BoldItalicMT
	Font: TimesNewRomanPSMT
	Font: TimesNewRomanPS-BoldMT
	Font: TimesNewRomanPS-ItalicMT
Family: Telugu Sangam MN
	Font: TeluguSangamMN
	Font: TeluguSangamMN-Bold
Family: Heiti SC
	Font: STHeitiSC-Medium
	Font: STHeitiSC-Light
Family: Apple SD Gothic Neo
	Font: AppleSDGothicNeo-Thin
	Font: AppleSDGothicNeo-SemiBold
	Font: AppleSDGothicNeo-Medium
	Font: AppleSDGothicNeo-Regular
	Font: AppleSDGothicNeo-Bold
	Font: AppleSDGothicNeo-Light
Family: Futura
	Font: Futura-Medium
	Font: Futura-CondensedMedium
	Font: Futura-MediumItalic
	Font: Futura-CondensedExtraBold
Family: Bodoni 72
	Font: BodoniSvtyTwoITCTT-Book
	Font: BodoniSvtyTwoITCTT-Bold
	Font: BodoniSvtyTwoITCTT-BookIta
Family: Baskerville
	Font: Baskerville-Bold
	Font: Baskerville-SemiBoldItalic
	Font: Baskerville-BoldItalic
	Font: Baskerville
	Font: Baskerville-SemiBold
	Font: Baskerville-Italic
Family: Symbol
	Font: Symbol
Family: Heiti TC
	Font: STHeitiTC-Medium
	Font: STHeitiTC-Light
Family: Copperplate
	Font: Copperplate
	Font: Copperplate-Light
	Font: Copperplate-Bold
Family: Party LET
	Font: PartyLetPlain
Family: American Typewriter
	Font: AmericanTypewriter-Light
	Font: AmericanTypewriter-CondensedLight
	Font: AmericanTypewriter-CondensedBold
	Font: AmericanTypewriter
	Font: AmericanTypewriter-Condensed
	Font: AmericanTypewriter-Bold
Family: Chalkboard SE
	Font: ChalkboardSE-Light
	Font: ChalkboardSE-Regular
	Font: ChalkboardSE-Bold
Family: Avenir Next
	Font: AvenirNext-MediumItalic
	Font: AvenirNext-Bold
	Font: AvenirNext-UltraLight
	Font: AvenirNext-DemiBold
	Font: AvenirNext-HeavyItalic
	Font: AvenirNext-Heavy
	Font: AvenirNext-Medium
	Font: AvenirNext-Italic
	Font: AvenirNext-UltraLightItalic
	Font: AvenirNext-BoldItalic
	Font: AvenirNext-Regular
	Font: AvenirNext-DemiBoldItalic
Family: Bangla Sangam MN
	Font: BanglaSangamMN
	Font: BanglaSangamMN-Bold
Family: Noteworthy
	Font: Noteworthy-Bold
	Font: Noteworthy-Light
Family: Zapfino
	Font: Zapfino
Family: Tamil Sangam MN
	Font: TamilSangamMN
	Font: TamilSangamMN-Bold
Family: Chalkduster
	Font: Chalkduster
Family: Arial Hebrew
	Font: ArialHebrew-Bold
	Font: ArialHebrew-Light
	Font: ArialHebrew
Family: Georgia
	Font: Georgia-BoldItalic
	Font: Georgia-Bold
	Font: Georgia-Italic
	Font: Georgia
Family: Helvetica Neue
	Font: HelveticaNeue-BoldItalic
	Font: HelveticaNeue-Light
	Font: HelveticaNeue-UltraLightItalic
	Font: HelveticaNeue-CondensedBold
	Font: HelveticaNeue-MediumItalic
	Font: HelveticaNeue-Thin
	Font: HelveticaNeue-Medium
	Font: HelveticaNeue-ThinItalic
	Font: HelveticaNeue-LightItalic
	Font: HelveticaNeue-UltraLight
	Font: HelveticaNeue-Bold
	Font: HelveticaNeue
	Font: HelveticaNeue-CondensedBlack
Family: Gill Sans
	Font: GillSans
	Font: GillSans-Italic
	Font: GillSans-BoldItalic
	Font: GillSans-Light
	Font: GillSans-LightItalic
	Font: GillSans-Bold
Family: Palatino
	Font: Palatino-Roman
	Font: Palatino-Italic
	Font: Palatino-Bold
	Font: Palatino-BoldItalic
Family: Courier New
	Font: CourierNewPSMT
	Font: CourierNewPS-BoldMT
	Font: CourierNewPS-ItalicMT
	Font: CourierNewPS-BoldItalicMT
Family: Oriya Sangam MN
	Font: OriyaSangamMN
	Font: OriyaSangamMN-Bold
Family: Didot
	Font: Didot-Bold
	Font: Didot-Italic
	Font: Didot
Family: DIN Alternate
	Font: DINAlternate-Bold
Family: Bodoni 72 Smallcaps
	Font: BodoniSvtyTwoSCITCTT-Book
```

字体如下图所示：

![UIFont](http://imsg.github.com/images/2013/UIFont_List.jpg)
