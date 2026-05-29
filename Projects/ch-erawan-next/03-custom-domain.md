# Custom Domain Setup Implementation Plan

## Overview
Configure ch-erawan.com domain to point to the Vercel deployment. This moves the site from `ch-erawanwebsite.vercel.app` to a branded domain.

**Complexity**: Low (30-60 minutes)  
**Priority**: Medium  
**Blockers**: Domain already purchased ✅ | Vercel project ready ✅

---

## Acceptance Criteria
- [ ] ch-erawan.com resolves to ch-erawanwebsite.vercel.app
- [ ] HTTPS certificate auto-issued by Vercel/Let's Encrypt
- [ ] All pages accessible via ch-erawan.com
- [ ] www.ch-erawan.com redirects to ch-erawan.com (or vice versa)
- [ ] No mixed content warnings (all assets HTTPS)
- [ ] DNS propagation confirmed globally
- [ ] Google Search Console updated with new domain
- [ ] Analytics (Google Analytics, Vercel Analytics) track ch-erawan.com traffic

---

## What Changes

### Vercel Configuration:
1. Go to Vercel Dashboard → ch-erawanwebsite project → Settings → Domains
2. Add `ch-erawan.com` as custom domain
3. Vercel will show DNS records to add to domain registrar

### Domain Registrar Configuration:
Locate where domain was purchased (GoDaddy, Namecheap, CloudFlare, etc.):

**If using Vercel's DNS (recommended):**
- Change nameservers to Vercel's:
  - NS1: ns1.vercel-dns.com
  - NS2: ns2.vercel-dns.com

**If using registrar DNS (A/CNAME records):**
- Add CNAME record:
  - Name: `ch-erawan`
  - Points to: `cname.vercel-dns.com`
- For www:
  - Name: `www`
  - Points to: `cname.vercel-dns.com`

---

## Implementation Notes

### Step 1: Verify Domain Ownership (5 min)
- Confirm you have access to domain registrar account
- Have admin email ready for domain verification

### Step 2: Configure Vercel (10 min)
1. Vercel Dashboard → Projects → ch-erawanwebsite
2. Settings → Domains
3. Add Domain → enter `ch-erawan.com`
4. Choose DNS provider option
5. Vercel generates DNS records

### Step 3: Update DNS Records (10-20 min)
- Log into domain registrar
- Navigate to DNS settings
- Add/update nameservers OR CNAME records per Vercel
- Save changes

### Step 4: Wait for Propagation (15-30 min)
- DNS propagation takes 2-48 hours (usually 5-15 min)
- Check status with:
  ```bash
  dig ch-erawan.com
  nslookup ch-erawan.com
  ```
- Vercel shows "Domain Added" when ready

### Step 5: Configure Redirects (5 min)
In Vercel → Settings → Redirects, add:
```
Source: /
Destination: https://ch-erawan.com
Type: Permanent (308)
```

This ensures `ch-erawanwebsite.vercel.app` → `ch-erawan.com`

### Step 6: Verify SSL/HTTPS (automatic)
- Vercel auto-issues Let's Encrypt certificate
- Takes 5-10 minutes after DNS resolves
- Green padlock appears once ready

---

## Success Metrics
✅ ch-erawan.com loads without 404  
✅ HTTPS certificate issued (no warnings)  
✅ All pages accessible (/, /contact, /booking, etc.)  
✅ www.ch-erawan.com redirects properly  
✅ Mixed content warnings: 0  
✅ DNS checks pass (nslookup, dig, whois)  

---

## Blockers/Dependencies
- ✅ ch-erawan.com purchased and accessible
- ✅ Vercel project deployed and working
- ⏳ Domain registrar access required

---

## Verification Checklist
- [ ] nslookup ch-erawan.com returns correct IP
- [ ] curl -I ch-erawan.com returns 200 OK
- [ ] https://ch-erawan.com has green padlock (SSL)
- [ ] Google Search Console can access site
- [ ] All assets load from https (no mixed content)
- [ ] Mobile site works properly
- [ ] Contact form works with ch-erawan.com domain

---

## Next Steps
1. Access domain registrar account
2. Follow Vercel DNS instructions
3. Wait for DNS propagation
4. Verify all checks pass
5. Update documentation (docs.sellsuki.com, etc.)
6. Monitor analytics for traffic from new domain
