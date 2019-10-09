---
layout: post
title:  "Tridion and Marketo integration basics "
date:   2019-01-23 16:15:05 +0200
categories: tridion
---

Marketo integration allows you to quickly and easily embed marketo forms on a website. 
From within the Marketo platform, you can insert a token into your email campaign to assign personalized survey links to each unique lead.
To embed marketo form on a tridion powered website we are going to use Forms 2.0 javascript api.
At the begining we need a form to embed on website. Let's create a form using Form Editor 2.0 from within Marketo Lead Management form designer.
You need to approve the form and get it embed code, in the result you should have script block like this one:

{% highlight html %}
    <script src="//app-sjqe.marketo.com/js/forms2/js/forms2.js"></script>
    <form id="mktoForm_621"></form>
    <script>
        MktoForms2.loadForm("//app-sjqe.marketo.com", "718-GIV-198", 621);
    </script>
{% endhighlight %}

Next, let's create tridion schema for the form component, with FormEmbedCode field that will contain our form script and a field that 
have a campaign info data for the form.

{% highlight xml %}
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="uuid:a1b13a5c-cd8b-474d-8f77-0d643856762b" xmlns:tcmi="http://www.tridion.com/ContentManager/5.0/Instance" elementFormDefault="qualified" targetNamespace="uuid:a1b13a5c-cd8b-474d-8f77-0d643856762b">
    <xsd:import namespace="http://www.tridion.com/ContentManager/5.0/Instance"></xsd:import>
    <xsd:include schemaLocation="tcm:2-8828-8"></xsd:include>
    <xsd:annotation>
        <xsd:appinfo>
        <tcm:Labels xmlns:tcm="http://www.tridion.com/ContentManager/5.0">
            <tcm:Label ElementName="FormEmbedCode" Metadata="false">Form embed code</tcm:Label>
            <tcm:Label ElementName="CampaignInfo" Metadata="false">CampaignInfo</tcm:Label>
        </tcm:Labels>
        </xsd:appinfo>
    </xsd:annotation>
    <xsd:element name="Content">
        <xsd:complexType>
        <xsd:sequence>
            <xsd:element name="FormEmbedCode" minOccurs="0" maxOccurs="1" type="tcmi:MultiLineText">
            <xsd:annotation>
                <xsd:appinfo>
                <ExtensionXml xmlns="http://www.tridion.com/ContentManager/5.0"></ExtensionXml>
                <tcm:Size xmlns:tcm="http://www.tridion.com/ContentManager/5.0">3</tcm:Size>
                </xsd:appinfo>
            </xsd:annotation>
            </xsd:element>
            <xsd:element name="CampaignInfo" minOccurs="0" maxOccurs="1" type="CampaignInfo">
            <xsd:annotation>
                <xsd:appinfo>
                <ExtensionXml xmlns="http://www.tridion.com/ContentManager/5.0"></ExtensionXml>
                <tcm:EmbeddedSchema xmlns:tcm="http://www.tridion.com/ContentManager/5.0" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="tcm:2-8828-8"></tcm:EmbeddedSchema>
                </xsd:appinfo>
            </xsd:annotation>
            </xsd:element>
        </xsd:sequence>
        </xsd:complexType>
    </xsd:element>
    </xsd:schema>
{% endhighlight %}

Field with campaign info is an embed schema:

{% highlight xml %}
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:tcmi="http://www.tridion.com/ContentManager/5.0/Instance" xmlns:category="tcm:0-2-1/Categories.xsd" elementFormDefault="qualified">
    <xsd:import namespace="http://www.tridion.com/ContentManager/5.0/Instance"></xsd:import>
    <xsd:import namespace="tcm:0-2-1/Categories.xsd"></xsd:import>
    <xsd:annotation>
        <xsd:appinfo>
        <tcm:Labels xmlns:tcm="http://www.tridion.com/ContentManager/5.0">
            <tcm:Label ElementName="CampaignId" Metadata="false">Campaign Id</tcm:Label>
            <tcm:Label ElementName="InterestProfile" Metadata="false">Interest Profile</tcm:Label>
        </tcm:Labels>
        </xsd:appinfo>
    </xsd:annotation>
    <xsd:complexType name="CampaignInfo">
        <xsd:sequence>
        <xsd:element name="CampaignId" minOccurs="0" maxOccurs="1" type="xsd:normalizedString">
            <xsd:annotation>
            <xsd:appinfo>
                <tcm:ExtensionXml xmlns:tcm="http://www.tridion.com/ContentManager/5.0"></tcm:ExtensionXml>
            </xsd:appinfo>
            </xsd:annotation>
        </xsd:element>
        <xsd:element name="InterestProfile" minOccurs="0" maxOccurs="1" type="category:InterestProfile">
            <xsd:annotation>
            <xsd:appinfo>
                <tcm:ExtensionXml xmlns:tcm="http://www.tridion.com/ContentManager/5.0"></tcm:ExtensionXml>
                <tcm:Size xmlns:tcm="http://www.tridion.com/ContentManager/5.0">1</tcm:Size>
                <tcm:listtype xmlns:tcm="http://www.tridion.com/ContentManager/5.0">tree</tcm:listtype>
            </xsd:appinfo>
            </xsd:annotation>
        </xsd:element>
        </xsd:sequence>
    </xsd:complexType>
    </xsd:schema>
{% endhighlight %}

Next step is to create a component based on Embed Form schema and a view model to render the component:

{% highlight xml %}
    <Content xmlns="uuid:a1b13a5c-cd8b-474d-8f77-0d643856762b">
        <FormEmbedCode>&lt;script src="//app-sjqe.marketo.com/js/forms2/js/forms2.min.js"&gt;&lt;/script&gt;
    &lt;form id="mktoForm_2320"&gt;&lt;/form&gt;
    &lt;script&gt;MktoForms2.loadForm("//app-sjqe.marketo.com", "718-GIV-198", 621);&lt;/script&gt;
    </FormEmbedCode>
        <CampaignInfo>
            <CampaignId>70160000000Bt1I</CampaignId>
            <InterestProfile xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="tcm:68-48881-1024" xlink:title="NoIP">NoIP</InterestProfile>
        </CampaignInfo>	
    </Content>
{% endhighlight %}


{% highlight CSharp %}
    public class MarketoFormViewModel : ViewModelBase, IMarketoFormViewModel
    {
        public string FormEmbedCode { get; set; }
        public List<string> CampaignsId { get; set; }
        public List<string> InterestProfiles { get; set; }

        public IDictionary<string, object> DataAttributes
        {
            get
            {
                IDictionary<string, object> attrs = new Dictionary<string, object>();
                attrs.Add(DataAttribute.Marketo.CampaignId, this.CampaignsId.ToCSV(",").ToUriString());
                attrs.Add(DataAttribute.Marketo.InterestProfile, this.InterestProfiles.ToCSV(",").ToUriString());
                return attrs;
            }
        }
    }
{% endhighlight %}

Once we are done with component and view model representation let's render it on a website:

{% highlight html %}
    <div @Html.ComponentAttributes() class="marketo-form__wrapper" @Html.HtmlAttributes(Model.DataAttributes)>
        @Html.Raw(Model.FormEmbedCode)
    </div>
  {% endhighlight %}

The the result of a manipulations that you made before, on a webpage, should look like:

{% highlight html %}
    <div class="marketo-form__wrapper" data-campaign-id="7010z0000019v17" data-interest-profile="NoIP">
        <script src="//app-sjqe.marketo.com/js/forms2/js/forms2.min.js"></script>  
        <form id="mktoForm_1966"></form>      
        <script>MktoForms2.loadForm("//app-sjqe.marketo.com", "689-GIV-525", 1966);</script>
    </div>
{% endhighlight %}

To pass correct data with marketo form submition, we need to add a small piece of javascript.
Marketo form api include MktoForms2.whenReady function that fire right after the marketo form is loaded.
By modifying callback function, we can add the code that will add required data to marketo request:

{% highlight javascript %}
    <script>
        if (typeof (MktoForms2) != "undefined") {
            MktoForms2.whenReady(function (form) {

                var frm = $(form.getFormElem());

                var parent = frm.parents('.marketo-form__wrapper');

                var vals = {
                    'ActivityId': (new Date().getTime().toString()),
                    'ActivityCampaignIds': parent.data('campaign-id'),
                    'ActivityInterestProfile': parent.data('interest-profile'),
                };

                form.setValues(vals);

                form.onSuccess(function (values, followUp) {
                    form = document.createElement('form');
                    form.action = action;
                    form.method = "POST";

                    document.body.appendChild(form);

                    form.submit();

                    return false;
                });

            });
        }
    </script>
{% endhighlight %}

This is basic example of how marketo form can be embedded on any page of your website.