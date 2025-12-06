using Unity.Netcode;
using UnityEngine;
using System.Collections.Generic;

public class TradeManager : NetworkBehaviour
{
    public static TradeManager Instance;

    // Aktif ticaret oturumları
    private Dictionary<ulong, TradeSession> activeTrades = new Dictionary<ulong, TradeSession>();
    private ulong nextSessionId = 1;

    public override void OnNetworkSpawn()
    {
        if (Instance == null)
            Instance = this;
        else
            Destroy(gameObject);
    }
    
    // SERVER-SIDE: Ticaret isteği başlatma
    [ServerRpc(RequireOwnership = false)]
    public void RequestTradeServerRpc(ulong requesterId, ulong targetId)
    {
        // 1. Oyuncuların müsait olup olmadığını kontrol et
        if (IsPlayerInTrade(requesterId) || IsPlayerInTrade(targetId))
        {
            // İstemcilere hata mesajı gönderilebilir
            return;
        }

        // 2. Yeni oturum oluştur
        ulong sessionId = nextSessionId++;
        TradeSession session = new TradeSession(sessionId, requesterId, targetId);
        activeTrades.Add(sessionId, session);

        // 3. İstemcilere ticaret daveti gönder (Target'a)
        ClientRpcParams targetParams = new ClientRpcParams { TargetClientIds = new ulong[] { targetId } };
        ReceiveTradeInviteClientRpc(sessionId, requesterId, targetParams);
    }
    
    // SERVER-SIDE: Ticaret teklifine eşya ekleme
    [ServerRpc(RequireOwnership = false)]
    public void AddItemToTradeServerRpc(ulong sessionId, ulong clientId, int itemIndex)
    {
        if (activeTrades.TryGetValue(sessionId, out TradeSession session))
        {
            if (session.requesterId == clientId)
            {
                session.RequesterItems.Add(itemIndex);
            }
            else if (session.targetId == clientId)
            {
                session.TargetItems.Add(itemIndex);
            }
            // Tüm istemcilere (hem requester hem target) tekliflerin güncellendiğini bildir
            UpdateTradeOfferClientRpc(sessionId, session.RequesterItems.ToArray(), session.TargetItems.ToArray());
        }
    }

    // SERVER-SIDE: Ticareti onaylama
    [ServerRpc(RequireOwnership = false)]
    public void ConfirmTradeServerRpc(ulong sessionId, ulong clientId)
    {
        if (activeTrades.TryGetValue(sessionId, out TradeSession session))
        {
            if (session.requesterId == clientId)
                session.requesterConfirmed = true;
            else if (session.targetId == clientId)
                session.targetConfirmed = true;
                
            // İki taraf da onayladıysa işlemi tamamla
            if (session.requesterConfirmed && session.targetConfirmed)
            {
                ExecuteTrade(session);
                activeTrades.Remove(sessionId);
            }
            else
            {
                 // Onay durumunun değiştiğini istemcilere bildir
                 UpdateConfirmationStatusClientRpc(sessionId, session.requesterConfirmed, session.targetConfirmed);
            }
        }
    }
    
    // SERVER-SIDE: Eşya takasını gerçekleştir ve envanterleri güncelle
    private void ExecuteTrade(TradeSession session)
    {
        // Gerçek bir uygulamada burada veritabanı işlemleri yapılır.
        
        // Örnek:
        // InventoryManager.Instance.TransferItems(session.requesterId, session.targetId, session.RequesterItems);
        // InventoryManager.Instance.TransferItems(session.targetId, session.requesterId, session.TargetItems);
        
        // İşlem tamamlandı mesajını istemcilere gönder
        TradeCompleteClientRpc(session.sessionId, true);
    }

    private bool IsPlayerInTrade(ulong clientId)
    {
        foreach (var session in activeTrades.Values)
        {
            if (session.requesterId == clientId || session.targetId == clientId)
                return true;
        }
        return false;
    }
    
    // İstemcilere ticaretin başladığını bildir
    [ClientRpc]
    private void ReceiveTradeInviteClientRpc(ulong sessionId, ulong inviterId, ClientRpcParams clientRpcParams)
    {
        // UI'ı aç: "Oyuncu X size ticaret daveti gönderdi. Kabul ediyor musunuz?"
        Debug.Log($"Ticaret Daveti: Oturum ID {sessionId}, Gönderen {inviterId}");
    }
    
    // İstemcilere tekliflerin değiştiğini bildir
    [ClientRpc]
    private void UpdateTradeOfferClientRpc(ulong sessionId, int[] requesterItems, int[] targetItems)
    {
        // UI'daki ticaret kutularını güncelle (Hangi eşyaların eklendiğini göster)
        Debug.Log($"Ticaret Teklifleri Güncellendi. İsteyen Eşya Sayısı: {requesterItems.Length}");
    }
    
    // İstemcilere onay durumunun değiştiğini bildir
    [ClientRpc]
    private void UpdateConfirmationStatusClientRpc(ulong sessionId, bool reqConfirmed, bool targetConfirmed)
    {
        // UI'da onay düğmelerinin durumunu güncelle (Örn: Yeşil tik göster)
        Debug.Log($"Onay Durumu: İsteyen={reqConfirmed}, Hedef={targetConfirmed}");
    }
    
    // İstemcilere ticaretin tamamlandığını bildir
    [ClientRpc]
    private void TradeCompleteClientRpc(ulong sessionId, bool success)
    {
        // Ticaret arayüzünü kapat ve sonucu göster
        Debug.Log($"Ticaret Tamamlandı. Başarı: {success}");
    }
}

// Ticaret oturumu veri sınıfı
public class TradeSession
{
    public ulong sessionId;
    public ulong requesterId;
    public ulong targetId;
    
    // Envanterdeki eşyaların ID'leri veya Index'leri
    public List<int> RequesterItems = new List<int>();
    public List<int> TargetItems = new List<int>();
    
    public bool requesterConfirmed = false;
    public bool targetConfirmed = false;
    
    public TradeSession(ulong sid, u req, ulong target)
    {
        sessionId = sid;
        requesterId = req;
        targetId = target;
    }
}
