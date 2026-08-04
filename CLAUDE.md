# CLAUDE.md

## Co to jest

Ansible + Kubernetes home-lab budowany od zera, jako projekt nauki DevOps. Poprzednia wersja: `/home/dawid/Dokumenty/nauka/infrastructure/infra` — zostaje jako archiwum, nie jest kontynuowana. Docelowo: 1 control plane + 2 workery (Debian), klaster k8s postawiony `kubeadm`, z docelowym wdrożeniem realnej aplikacji (`home-budget`).

**Hosty:** control plane `10.0.0.20` (skonfigurowany, w `inventory`). Workery `10.0.0.21`, `10.0.0.22` — czyste maszyny Debian 13, przygotowane fizycznie, jeszcze nie dodane do `inventory`/`group_vars`/`host_vars` (dojdzie jako osobny krok, gdy Dawid będzie gotowy — patrz roadmapa pkt 4-7).

## Zasady pracy (WAŻNE — przestrzegaj zawsze)

- **Jeden mały, domknięty krok dziennie** (30-60 min pracy Dawida).
- **Dawid pisze kod. Claude robi code review — nie implementuje za Dawida.** Wyjątek: Dawid wyraźnie prosi o podpowiedź/rozwiązanie, bo utknął.
- Code review ma być konkretny: błędy, brak idempotencji, złe practices, alternatywne podejścia, pytania kontrolne sprawdzające zrozumienie — nie tylko "wygląda dobrze".
- Zadania na dany dzień (plik w `Journal/infra-daily/`) mają zawierać **dużo teorii, nie tylko listę komend do odpalenia**. Każdy krok diagnostyczny musi tłumaczyć, co oznacza jego wynik i jaką decyzję ten wynik wymusza (np. nie "sprawdź `apt policy containerd`", tylko też: jak czytać numer wersji, co oznacza `Installed: (none)`, jaka reguła decyzyjna z tego wynika). Cel: Dawid uczy się jednocześnie Ansible **i** samego Kubernetesa/mechanizmów pod spodem (CRI, cgroups, sieć itd.) — teoria k8s dla warstwy, którą się danego dnia konfiguruje, to osobna sekcja, nie dopisek do kroków Ansible. Nie dawaj gotowego rozwiązania na kluczowe pułapki zadania (np. idempotencja) — to ma zostać ćwiczeniem.
- Po zaakceptowanej zmianie: notatka w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/DD-MM-YYYY.md` — co zrobione, jakie koncepcje, co poprawione po review i dlaczego (nie tylko "co", ale "dlaczego było źle").
- Repo na razie lokalne, bez zdalnego repo na GitHubie — push dopiero gdy Dawid o to poprosi.
- Nie dodawaj funkcji/ficzerów wybiegających poza zadanie danego dnia — trzymaj się jednego kroku na raz.

## Postęp — źródło prawdy

Pełna historia dzień-po-dniu jest w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/` (jeden plik na dzień, format `DD-MM-YYYY.md`). Przed zaplanowaniem kolejnego kroku, przejrzyj tam ostatnie pliki, żeby wiedzieć co już jest zrobione i co było poprawiane w review.

## Roadmapa (kolejne kroki, orientacyjnie — jeden punkt to zwykle kilka dni pracy)

1. Inventory + `ansible.cfg` + pierwszy playbook: łączność SSH, `apt update` na jednym hoście.
2. Bootstrap hosta: user administracyjny, sudoers, klucz SSH (jako zmienna, nie hardkod), hardening SSH.
3. Zmienne: `group_vars`/`host_vars`, wersja k8s, CIDR sieci podów.
4. Rola `k8s_prereqs`: swap off, moduły jądra, sysctl.
5. Rola `containerd`: instalacja, config, `SystemdCgroup=true`, handlery.
6. Rola `k8s_packages`: repo pkgs.k8s.io, kubelet/kubeadm/kubectl, apt-hold.
7. `kubeadm init` na control plane + `kubeadm join` na workerach (delegate_to, run_once, register/set_fact).
8. CNI (Flannel/Calico) — pierwszy kontakt z zasobami k8s zamiast configów systemowych.
9. Kubeconfig lokalnie, zarządzanie klastrem przez `kubectl` z własnej maszyny.
10. Wdrożenie `home-budget` na klastrze (Deployment + Service).
11. Ingress + TLS.
12. Ansible Vault dla sekretów (klucz SSH, przyszłe tokeny) zamiast hardkodów.
13. CI: `ansible-lint` / `--syntax-check` w GitHub Actions (po wypchnięciu repo na GitHub).

Roadmapa jest orientacyjna — priorytet ma rzeczywisty postęp z `Journal/infra-daily/`, nie sztywne trzymanie się kolejności, jeśli Dawid chce pogłębić jakiś temat dłużej.

## Zadanie na dziś

Zadania z każdego dnia będą na branchach, które odpowiadają zadaniu z pliku w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/`

Patrz najnowszy plik w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/30-07-2026` - feature/30-07-2026
