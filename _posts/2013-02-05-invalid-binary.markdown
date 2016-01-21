---
layout: page
title:  Possible reason of “Invalid binary” ITunes Connect error (with Unity3D app)
date:   2013-02-05 12:00:00
---

That was a nightmare for me recently to figure out why I get this error when submitting my app to the ITunes. The text I got by email was the following:
> Invalid Signature – the main app bundle Uncopy at path Uncopy.app is signed but the signature is invalid. The following error(s) were reported from codesign:
> a sealed resource is missing or invalid
> In architecture: i386

Thanks to [this question](http://stackoverflow.com/questions/13814506/mac-store-invalid-binary-on-itunesconnect-the-signature-is-invalid) (see the second and third comments), the reason was a file named `._unity default resources` that Unity put into the `<App bundle>.app/Resources` folder.
The solution for me was simple, I just deleted that file, re-signed the app and re-submitted, with no errors. I have no clue what’s that file for,
but my app launches well without it. Big thanks to the guy who posted that question and answer.