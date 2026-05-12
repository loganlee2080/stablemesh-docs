# 📋 Milestone: Mayer

## Goal
- [ ] Open world card for ADS users
- [ ] Make Revo card x3, max to 500 cards
- [ ] Global USD account support
- [ ] Enhance TRON deposit with auto mode

---

## 🚀 Execution Plan

### Phase 1: ADS World Card Integration (Week 1-2)

**1.1 API Integration**
- [ ] Verify legacy WorldFirst card code compatibility
- [ ] Integrate API key for live transaction processing
- [ ] Implement card balance and transaction query endpoints

**1.2 Card Pool Architecture**
- [ ] Design batch card open/sync mechanism with API rate limiting
- [ ] Implement card pool pattern (similar to EtherFi):
  - [ ] Pre-provision card pool with buffer capacity
  - [ ] Automatic card rotation and replenishment
  - [ ] Exponential backoff for rate limit errors
- [ ] Add monitoring for API quota consumption and pool health

**1.3 Multi-Currency Card Infrastructure**
- [ ] Database schema updates: add currency_type enum to card table (USD, EUR, GBP, etc.)
- [ ] Currency conversion logic for multi-currency transactions

---

### Phase 2: Global USD Account Support (Week 2)

**2.1 Account Integration**
- [ ] Reference: [GitHub Issue #9](https://github.com/StableMesh/api-server/issues/9)
- [ ] Implement global USD account creation and management
- [ ] Add account balance and statement retrieval
- [ ] Support multi-currency transaction routing

---

### Phase 3: TRON Deposit Enhancement (Week 2)

**3.1 TRON Address Pool**
- [ ] Implement TRON deposit address pool for improved throughput
- [ ] Automatic address rotation and recycling
- [ ] Pool health monitoring and replenishment logic
- [ ] Deposit tracking across pooled addresses

**3.2 Auto Mode**
- [ ] Enable automatic deposit address assignment
- [ ] Real-time deposit confirmation and notification

---

## Dependencies

| Dependency | Owner | Status |
|------------|-------|--------|
| WorldFirst API access | Admin | Pending |
| Global USD provider integration | Admin | See #9 |
| TRON network reliability | External | Known |
