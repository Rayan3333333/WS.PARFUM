import { Card, CardContent } from "@/components/ui/card"; 
import { Button } from "@/components/ui/button";
import { useEffect, useState, useCallback, useRef } from "react";

const initialPerfumes = [
  { id: 1, name: "Yara", price: 10, image: "/images/essence-noire.jpg" },
  { id: 2, name: "Rose Vanille", price: 10, image: "/images/blanche-elegance.jpg" },
  { id: 3, name: "OUD Madawi", price: 10, image: "/images/mystere-intense.jpg" },
];

export default function WSParfum() { // Changer le nom de la fonction ici
  const [cart, setCart] = useState(() => {
    const savedCart = localStorage.getItem("cart");
    return savedCart ? JSON.parse(savedCart) : [];
  });

  const [perfumes, setPerfumes] = useState(() => {
    const saved = localStorage.getItem("perfumes");
    return saved ? JSON.parse(saved) : initialPerfumes;
  });

  const paypalRef = useRef();
  const applePayRef = useRef();
  const cardRef = useRef();

  useEffect(() => {
    localStorage.setItem("perfumes", JSON.stringify(perfumes));
    localStorage.setItem("cart", JSON.stringify(cart));
  }, [perfumes, cart]);

  useEffect(() => {
    if (cart.length > 0) {
      const paypalScript = document.createElement("script");
      paypalScript.src = "https://www.paypal.com/sdk/js?client-id=sb&currency=EUR";
      paypalScript.addEventListener("load", () => {
        if (window.paypal && paypalRef.current) {
          window.paypal.Buttons({
            createOrder: (data, actions) => {
              return actions.order.create({
                purchase_units: [{
                  amount: {
                    value: cart.reduce((sum, item) => sum + item.price, 0).toFixed(2),
                  },
                }],
              });
            },
            onApprove: (data, actions) => {
              return actions.order.capture().then((details) => {
                alert("Paiement effectué par " + details.payer.name.given_name);
                setCart([]);
              });
            },
            onError: (err) => {
              console.error(err);
              alert("Une erreur est survenue pendant le paiement.");
            },
          }).render(paypalRef.current);
        }
      });
      document.body.appendChild(paypalScript);

      if (window.ApplePaySession && ApplePaySession.canMakePayments()) {
        const appleButton = document.createElement("button");
        appleButton.textContent = "Payer avec Apple Pay";
        appleButton.className = "bg-black text-white py-2 px-4 rounded w-full mt-4";
        appleButton.onclick = () => {
          const paymentRequest = {
            countryCode: "FR",
            currencyCode: "EUR",
            supportedNetworks: ["visa", "masterCard", "amex"],
            merchantCapabilities: ["supports3DS"],
            total: {
              label: "WS.PARFUM", // Garder "WS.PARFUM" ici
              amount: cart.reduce((sum, item) => sum + item.price, 0).toFixed(2),
            },
          };

          const session = new ApplePaySession(3, paymentRequest);

          session.onvalidatemerchant = (event) => {
            session.completeMerchantValidation({});
          };

          session.onpaymentauthorized = (event) => {
            session.completePayment(ApplePaySession.STATUS_SUCCESS);
            alert("Paiement Apple Pay réussi");
            setCart([]);
          };

          session.begin();
        };
        if (applePayRef.current) {
          applePayRef.current.innerHTML = "";
          applePayRef.current.appendChild(appleButton);
        }
      }

      if (cardRef.current) {
        cardRef.current.innerHTML = `
          <div class="mt-4">
            <label class="block mb-2">Paiement par carte bancaire (demo)</label>
            <input type="text" placeholder="Numéro de carte" class="w-full p-2 mb-2 rounded text-black" />
            <input type="text" placeholder="MM/AA" class="w-1/2 p-2 mb-2 mr-2 rounded text-black" />
            <input type="text" placeholder="CVC" class="w-1/3 p-2 mb-2 rounded text-black" />
            <button class="bg-blue-600 text-white py-2 px-4 rounded w-full" onclick="alert('Paiement carte bancaire simulé');">Payer</button>
          </div>
        `;
      }
    }
  }, [cart]);

  const addToCart = useCallback((perfume) => {
    setCart((prev) => {
      const existing = prev.find((item) => item.id === perfume.id);
      if (existing) return prev;
      return [...prev, perfume];
    });
  }, []);

  const removeFromCart = useCallback((perfumeId) => {
    setCart((prev) => prev.filter((item) => item.id !== perfumeId));
  }, []);

  const handleImageUpload = (e, perfumeId) => {
    const files = Array.from(e.target.files);
    files.forEach((file) => {
      if (file) {
        const reader = new FileReader();
        reader.onloadend = () => {
          setPerfumes((prevPerfumes) =>
            prevPerfumes.map((perfume) =>
              perfume.id === perfumeId ? { ...perfume, image: reader.result } : perfume
            )
          );
        };
        reader.readAsDataURL(file);
      }
    });
  };

  const total = cart.reduce((sum, item) => sum + item.price, 0);

  return (
    <main className="min-h-screen bg-black bg-cover bg-center text-white p-4 relative">
      <div className="absolute inset-0 flex items-center justify-center pointer-events-none select-none text-white text-[20vw] font-bold opacity-10">
        W S
      </div>

      <header className="text-center py-10 relative z-10">
        <h1 className="text-4xl font-bold text-white drop-shadow-lg tracking-wide">
          <span className="text-pink-500">W</span>
          <span className="text-blue-400">S</span>.PARFUM
        </h1>
        <p className="text-sm mt-2 text-gray-300">Parfums à 10€ - Livraison rapide - Paiement en 1 fois</p>
      </header>

      <section className="grid grid-cols-1 md:grid-cols-3 gap-6 max-w-5xl mx-auto relative z-10">
        {perfumes.map((perfume) => (
          <div key={perfume.id}>
            <Card className="bg-white text-black rounded-2xl shadow-xl">
              <CardContent className="p-6">
                {perfume.image ? (
                  <div className="mb-6 w-full h-48 overflow-x-auto whitespace-nowrap space-x-2 flex">
                    <img
                      src={perfume.image}
                      alt={perfume.name}
                      className="rounded-xl h-48 w-auto object-cover"
                    />
                  </div>
                ) : (
                  <div className="mb-6 w-full h-48 bg-gray-200 flex items-center justify-center text-gray-500 rounded-xl">
                    Aucune image
                  </div>
                )}
                <input
                  type="file"
                  accept="image/*"
                  onChange={(e) => handleImageUpload(e, perfume.id)}
                  className="mb-4 text-sm"
                />
                <h2 className="text-xl font-semibold mb-2 mt-2">{perfume.name}</h2>
                <p className="text-lg">{perfume.price}€</p>
                <Button className="mt-4 w-full" onClick={() => addToCart(perfume)}>
                  Ajouter au panier
                </Button>
              </CardContent>
            </Card>
          </div>
        ))}
      </section>

      <section className="max-w-3xl mx-auto mt-10 text-white relative z-10">
        <h2 className="text-2xl font-bold mb-4">Panier</h2>
        {cart.length === 0 ? (
          <p className="text-gray-400">Votre panier est vide.</p>
        ) : (
          <>
            <ul className="space-y-2">
              {cart.map((item, index) => (
                <li key={index} className="border-b border-gray-700 pb-2">
                  {item.name} - {item.price}€
                  <button
                    className="ml-4 text-red-500"
                    onClick={() => removeFromCart(item.id)}
                  >
                    Supprimer
                  </button>
                </li>
              ))}
            </ul>
            <p className="mt-4 font-semibold">Total : {total}€</p>
            <div ref={paypalRef} className="mt-6"></div>
            <div ref={applePayRef} className="mt-4"></div>
            <div ref={cardRef} className="mt-4"></div>
          </>
        )}
      </section>

      <section className="max-w-3xl mx-auto mt-20 text-gray-200 relative z-10">
        <h2 className="text-xl font-semibold mb-4">Avis clients</h2>
        <div className="space-y-6">
          <div className="bg-gray-800 p-4 rounded-xl">
            <p className="font-semibold">Fatima B.</p>
            <p className="text-sm text-gray-400">“J'ai été agréablement surprise par la qualité du parfum pour ce prix. Livraison rapide en plus !”</p>
          </div>
          <div className="bg-gray-800 p-4 rounded-xl">
            <p className="font-semibold">Julien L.</p>
            <p className="text-sm text-gray-400">“Excellent rapport qualité/prix. J’ai commandé pour offrir et tout le monde a adoré.”</p>
          </div>
          <div className="bg-gray-800 p-4 rounded-xl">
            <p className="font-semibold">Nadia R.</p>
            <p className="text-sm text-gray-400">“Les senteurs sont envoûtantes et le site est facile à utiliser. Je recommande !”</p>
          </div>
        </div>
      </section>

      <section className="max-w-3xl mx-auto mt-20 text-gray-200 relative z-10">
        <h2 className="text-xl font-semibold mb-2">À propos de WS.PARFUM</h2>
        <p className="mb-6">
          WS.PARFUM propose une sélection de parfums raffinés à prix accessibles. Notre mission : offrir le luxe à tous.
        </p>

        <h2 className="text-xl font-semibold mb-2">Livraison & Retours</h2>
        <p className="mb-6">
          Livraison rapide partout en France. Retours acceptés sous 14 jours après réception si le produit est intact.
        </p>

        <h2 className="text-xl font-semibold mb-2">Contact</h2>
        <p>Email : rayanfeddal1@gmail.com </p>
      </section>

      <footer className="text-center text-gray-400 mt-20 text-sm border-t border-gray-700 pt-6 relative z-10">
        © 2025 WS.PARFUM - Livraison partout en France - Paiement sécurisé<br />
        Nom de domaine : <span className="text-white">WS.parfum.fr</span>
      </footer>
    </main>
  );
}

