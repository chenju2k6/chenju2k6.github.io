---
title: aflplusplus
---


{% capture template %}



<div class="section">
    <h1>aflplusplus</h1>
    <p>
        This page shows the distribution of time-to-bug measurements for every bug reached and/or triggered by the
        fuzzer. The results are grouped by target to highlight any performance trends the fuzzer may have against
        specific targets.
    </p>

    
    <h2>openssl</h2>
    
        
        <h3>asn1</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_asn1_reached.svg">
            </div>
        
        </div>
    
        
        <h3>asn1parse</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_asn1parse_reached.svg">
            </div>
        
        </div>
    
        
        <h3>bignum</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_bignum_reached.svg">
            </div>
        
        </div>
    
        
        <h3>client</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_client_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_client_triggered.svg">
            </div>
        
        </div>
    
        
        <h3>server</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_server_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_server_triggered.svg">
            </div>
        
        </div>
    
        
        <h3>x509</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_x509_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_openssl_x509_triggered.svg">
            </div>
        
        </div>
    

    
    <h2>php</h2>
    
        
        <h3>exif</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_php_exif_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_php_exif_triggered.svg">
            </div>
        
        </div>
    

    
    <h2>poppler</h2>
    
        
        <h3>pdf_fuzzer</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdf_fuzzer_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdf_fuzzer_triggered.svg">
            </div>
        
        </div>
    
        
        <h3>pdfimages</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdfimages_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdfimages_triggered.svg">
            </div>
        
        </div>
    
        
        <h3>pdftoppm</h3>
        <div class="row">
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdftoppm_reached.svg">
            </div>
        
            
            <div class="col s6">
                <img class="materialboxed responsive-img" src="../plot/box_aflplusplus_poppler_pdftoppm_triggered.svg">
            </div>
        
        </div>
    

</div>



{% endcapture %}
{{ template | replace: '    ', ''}}
